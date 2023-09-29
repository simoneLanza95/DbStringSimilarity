# String Similarity Function and Database Components

## Description

This repository contains SQL functions and database components designed for calculating string similarity between names in a SQL Server database. The primary goal of these components is to efficiently find similar names within a large dataset and help identify potential matches.

## String Similarity Functions

### CalculateWordJaccardSimilarity

This function calculates the Jaccard similarity between two input words. It returns an integer representing the number of different characters between the words. It's used as a building block for more complex similarity calculations.

### CalculateSimilarity

The CalculateSimilarity function uses the CalculateWordJaccardSimilarity function to calculate a similarity score between a given input word and names in the database. It returns a floating-point value between 0 and 1, where 0 indicates no similarity, and 1 indicates an exact match. The threshold parameter allows you to filter results based on similarity.

## Database Components

### Customers Table

The `Customers` table is the primary dataset for name comparisons. It contains 50+ million records so I've decided to create the `Customers_Partitioned` table with an additional column, `ThreeLetter`, that is filled with the first three letters of the `name` column. 
Using partition function and scheme on the ThreeLetter column the new `Customers_Partitioned` table will be correcly divided in partitions based on a range of combinations of three letters.

### Partition Function and Scheme

The partition function and scheme are essential components of our database architecture. They enable us to:

- Improve Query Performance: Partitioning the table based on specific criteria, such as the first three letters of names, significantly enhances query performance when searching for similar names.

- Optimize Data Storage: By dividing the data into manageable partitions, we can optimize data storage and maintenance processes. Each partition is stored separately, allowing for efficient storage management.

- Streamline Data Loading: When new data is added to the Customers table, the partitioning scheme ensures that it is loaded into the appropriate partition, reducing data loading times.

In summary, the partition function and scheme play a crucial role in managing and querying our extensive dataset, allowing us to perform efficient string similarity calculations and find potential matches among millions of customer records.

### Indexes

A nonclustered columnstore index is created on the `name` column of the `Customers_Partitioned` table. Columnstore indexes are well-suited for large datasets and analytical workloads. They can help accelerate queries that involve string similarity calculations.
I've also used a nonclustered index on `ThreeLetter` column that includes the `name` column and other indexes if there's the need to retrieve other infos about the result records.

## Usage

You can use the provided SQL functions, such as `CalculateSimilarity`, to calculate string similarity between an input word and names in the database. The partitioning strategy and columnstore index should optimize query performance for these operations.

## Example Queries

Here is an example query to get you started:

```sql
-- Calculate similarity between 'inputWord' and names starting with 'XYZ' with a threshold of 4
SELECT name, dbo.CalculateSimilarity('inputWord', name, 4) AS similarity
FROM dbo.Customers_Partitioned
WHERE dbo.Customers_Partitioned.ThreeLetter = 'XYZ'
ORDER BY similarity DESC
