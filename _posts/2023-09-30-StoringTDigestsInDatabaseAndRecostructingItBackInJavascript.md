---
title: Storing TDigests In A Database And Reconstructing It Back In Javascript
description: Storing TDigests In A Database And Reconstructing It Back In Javascript
date: '2023-09-30'
categories: [Programming]
keywords: [TDigest, Node, Algorithms, SQLServer]
tags: [TDigest, Node, Javascript, Algorithms, SQLServer]
image:
  path: /assets/img/posts/percentiles.jpg
  width: 800
  height: 500
  alt:
---

[T-Digest](https://www.sciencedirect.com/science/article/pii/S2665963820300403) is a statistical algorithm used for approximate calculation of quantiles and percentiles from large data sets. It's particularly useful when you have a vast amount of data and you want to quickly estimate values like the median, quartiles, or any other percentile without having to process the entire dataset. Once the large dataset is processed the data will be added to the TDigest and from the Digest we are able to estimate the quantiles. In some cases, you may want to persist the TDigest so you can use that data at a different point. In this post, we will explore how we can persist the tidest we create with Nodejs in SQLServer and then reconstruct it.

Before we go into the solution it's important to understand Centroids in TDigest. In essence, centroids in T-Digest serve as central values or representatives of quantile ranges within the data distribution. We will be using these Centroids when storing the Digest in the DB.

Once we create the Digest in Javascript, the main problem with the TDigest object that we create is, that it's not serializable, there isn't an inbuilt mechanism in the TDigest implementation(Javascript version) to serialize the data out of the box. So as a workaround, we will be extracting all the Centroids from the digest and then storing it in the DB which will allow us to reconstruct the Digest from the DB.

Okay, let's go to the solution.

### Prerequisites.

For this example, I'll be using SQLServer and NodeJS. So these are the only dependencies. Then of course we need the TDigest [npm module](https://github.com/welch/tdigest) for this. Assuming you have `node` setup you can simply use `npm` to install the dependencies.

```sh
npm install tdigest mssql
```

Then let's create a simple Database and a table to store the data. For this, I'm using the SQLServer Docker image so I can quickly set up a DB locally to do the implementation etc.

Let's start the docker image.
```sh
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=Root#Pass" -p 1433:1433 -d mcr.microsoft.com/mssql/server:2022-latest
```

Then let's create a simple table to store the data. I have a column named `digest` with the type `varbinary` to store the digest as binary data.  

```sql
CREATE TABLE TDigests (
	id INT IDENTITY(1,1) PRIMARY KEY,
	digest varbinary(MAX) NOT NULL,
);
```

### Storing the data

Now we need to construct a TDigest and store it in the database. 

The following code will generate a digest and add 100,000 entries to it.

```javascript
const digest = new TDigest();

// Add 100000 random records to the T-Digest
for (let i = 0; i < 100000; i++) {
    digest.push(getRandomInt(10, 3000));
}
digest.compress();
console.log("90th Percentile before saving: " + digest.percentile(0.9));
```

Once the digest is created we can use the `digest.toArray()` as in [here](https://github.com/welch/tdigest/blob/cf73bdd0544c4b0b5e9b34a423e888c3c5b26413/tdigest.js#L46) to extract all the centroids. Once the centroid array is retrieved you can use the `JSON.stringify()` method to convert the Javascript object to a JSON object, which will allow us to store the data as a JSON object in the database. 

Now let's create a prepared statement and insert the data into the Database. 

```javascript
const ps = new sql.PreparedStatement(pool);
ps.input('digest', sql.NVarChar); // When we are storing the data we will compress it

await ps.prepare(`INSERT INTO ${tableName} VALUES (COMPRESS(@digest))`);
await ps.execute({ 'digest': JSON.stringify(digest.toArray()) });
```
In the above insert statement note the `COMPRESS()` SQLServer function which allows us to convert the JSON string into binary content before storing it in the DB. This will allow us to save a considerable amount of space in the DB. Also, compression is not mandatory. You can do the compression at the code level as well if that's what you prefer. 

### Retreiving the data

Once you insert the data you can retrieve the data using the following code.

```javascript
const getDigestQuery = `SELECT CAST(DECOMPRESS(digest) AS NVARCHAR(MAX)) AS digest 
                            FROM ${tableName}`;

const result = await pool.query(getDigestQuery)
```
Note when reading the digest we have to decompress it and convert it to a String so we can pass it back to a JSON. Once that is done we can use the following code the recontruct the digest back. For that, we can use `digest.push_centroid()` method as in [here](https://github.com/welch/tdigest/blob/cf73bdd0544c4b0b5e9b34a423e888c3c5b26413/tdigest.js#L93). 

```javascript
result.recordset.forEach(element => {
        let newDigest = new TDigest();
        newDigest.push_centroid(JSON.parse(element.digest));
        console.log("90th Percentile after constructing: " + newDigest.percentile(0.9));
    });
```

So that's it. The complete code is something like below. 

```javascript
const sql = require('mssql');
const { TDigest } = require('tdigest');

async function main() {

    SQL_CONFIG.user = "SA";
    SQL_CONFIG.password = "Root#Pass";
    SQL_CONFIG.server = "localhost";
    SQL_CONFIG.port = 1433;
    SQL_CONFIG.database = 'master'

    const tableName = "TDigests";

    let pool = await getDBPool();

    // Initialize a T-Digest data structure
    const digest = new TDigest();

    // Add 100000 random records to the T-Digest
    for (let i = 0; i < 100000; i++) {
        digest.push(getRandomInt(10, 3000));
    }
    digest.compress();
    console.log("90th Percentile before saving: " + digest.percentile(0.9));
    const ps = new sql.PreparedStatement(pool);
    ps.input('digest', sql.NVarChar); // When we are storing the data we will compress it

    await ps.prepare(`INSERT INTO ${tableName} VALUES (COMPRESS(@digest))`);
    await ps.execute({ 'digest': JSON.stringify(digest.toArray()) });

    console.log("Ok we are done saving the digest, Now let's read and construct it back")

    const getDigestQuery = `SELECT CAST(DECOMPRESS(digest) AS NVARCHAR(MAX)) AS digest 
                            FROM ${tableName}`;

    const result = await pool.query(getDigestQuery);

    result.recordset.forEach(element => {
        let newDigest = new TDigest()
        newDigest.push_centroid(JSON.parse(element.digest));
        console.log("90th Percentile after constructing: " + newDigest.percentile(0.9));
    });

    ps.unprepare();
    pool.close();
}

// Function to generate a random integer between min and max (inclusive)
function getRandomInt(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
}

const SQL_CONFIG = {
    user: '',
    password: '',
    database: '',
    server: '',
    port: 1433,
    pool: {
      max: 10,
      min: 0,
      idleTimeoutMillis: 60000
    },
    options: {
      encrypt: false,
      trustServerCertificate: false,
      trustedConnection: false,
    }
  }
  
  function getDBPool() {
    const poolPromise = new sql.ConnectionPool(SQL_CONFIG)
    .connect()
    .then(pool => {
      console.log('Connected to MSSQL');
      return pool
    })
    return poolPromise;
  }

if (require.main === module) {
    main();
}
``` 
You can find the full code at [Github](https://github.com/yasassri/tdigest-persist-sample) as well. 

Hope the above helps someone. Please drop a comment if you have any questions. 
