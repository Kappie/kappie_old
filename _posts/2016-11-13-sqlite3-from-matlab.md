---
layout: post
title: Storing MATLAB data using sqlite3
---

The usual way to store data in MATLAB is with the `save` command. But if you have data with many different parameters,
querying `.mat` files becomes cumbersome. What actually works, even with complex queries, is a database and a query language like
[SQL](https://en.wikipedia.org/wiki/SQL) (structured query language).  

SQL is a declarative language, meaning you specify what data you want, not how you want to get it. That means SQL takes
care of all look-up algorithms. We first create a database from the command line as follows:

{% highlight bash %}
# enter sqlite3 interpreter
sqlite3

sqlite> .open my_database.db

{% endhighlight %}

Suppose we want to store a `tensor` at a certain `temperature`, bond dimension `chi` and
`convergence`. We create a table as follows:

{% highlight sql %}
CREATE TABLE tensors (tensor BLOB, temperature NUMERIC, chi NUMERIC, convergence NUMERIC);
{% endhighlight %}

The `tensor` field is a `BLOB`, meaning it's raw bytes -- we'll have to write a function to serialize tensors from MATLAB in
this way. The other fields are `NUMERIC`, which does not require conversion from MATLAB numbers. The next step is adding data and querying the database, but this has to be
done from MATLAB. (In principle, the `CREATE TABLE` statement can be done from MATLAB too, but it is good to be familiar
with the sqlite interpreter.) Use `.exit` to exit the interpreter.

I use the package [`matlab-sqlite3-driver`](https://github.com/kyamagu/matlab-sqlite3-driver) for interfacing with SQL from MATLAB. Suppose we want to store this data:

{% highlight matlab %}
tensor = rand(2, 2, 2);
temperature = 3;
chi  = 16;
convergence = 1e-7;
{% endhighlight %}

First, we have to serialize `tensor`. That can be done as follows:

{% highlight matlab %}
function byte_stream = serialize(tensor)
  byte_stream = getByteStreamFromArray(tensor);
end
{% endhighlight %}

Now, we store the data by accessing the database and constructing a SQL query:

{% highlight matlab %}
database_id = sqlite3.open('my_database.db');
query = 'INSERT INTO tensors (tensor, temperature, chi, convergence) VALUES (?, ?, ?, ?)';
% Pass an argument for each '?' in query.
sqlite3.execute(database_id, query, getByteStreamFromArray(tensor), temperature, chi, convergence);
{% endhighlight %}

At a later time, we can query the database to obtain a struct with our data.

{% highlight matlab %}
database_id = sqlite3.open('my_database.db');
% '*' selects all four columns
query = 'SELECT * from tensors where temperature = ? AND chi = ? AND convergence = ?';
% Pass an argument for each '?' in query.
result = sqlite3.execute(database_id, query, temperature, chi, convergence);
tensor = deserialize(result.tensor);
{% endhighlight %}

With

{% highlight matlab %}
function tensor = deserialize(byte_stream)
  tensor = getArrayFromByteStream(byte_stream);
end
{% endhighlight %}

After the initial learning curve, it's very easy to construct queries like:

{% highlight sql %}
  SELECT * from tensors
  WHERE temperature = ? AND chi = ?
  AND convergence >= ?
  ORDER BY convergence ASC
  LIMIT 1
{% endhighlight %}

which selects the tensor with matching `temperature` and `chi`, but with a smallest available `convergence` bigger or equal to the one specified.

A final important issue is query optimalization. The above query does a full table scan to get its result, which quickly gets painful. This \\( \mathcal{O}(N) \\) look-up time can be reduced to \\( \mathcal{O}(k \log N) \\), where \\( N \\) is the number of rows and \\( k \\) the number of matching convergences, by adding an index:

{% highlight sql %}
  CREATE INDEX index_convergence
  ON tensors (temperature, chi, convergence);
{% endhighlight %}

On my machine, this reduced the running time of 500 queries from a table with \\( 10^5 \\) rows of random data from 12 seconds to 0.2 seconds.
