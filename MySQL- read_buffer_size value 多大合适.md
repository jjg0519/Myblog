# MySQL: what read_buffer_size value is optimal ?

[Peter Zaitsev](https://www.percona.com/blog/author/admin/)  | September 17, 2007 |  Posted In: [Benchmarks](https://www.percona.com/blog/category/benchmarks/), [Insight for DBAs](https://www.percona.com/blog/category/dba-insight/)

The more I work with MySQL Performance Optimization and Optimization for other applications the better I understand I have to less believe in common sense or common sense of documentation writers and do more benchmarks and performance research. I just recently wrote about rather surprising results with [sort performance](https://www.percona.com/blog/2007/08/18/how-fast-can-you-sort-data-with-mysql/) and today I’ve discovered even **read_buffer_size** selection may be less than obvious.

What do we generally hear about **read_buffer_size** tuning ? If you want fast full table scans for large table you should set this variable to some high value. Sample my.cnf values on large memory sizes recommend 1M settings and MySQL built-in default is 128K. Some people having a lot of memory and few concurrent connections set it as high as 32M in hopes for better performance. Lets see if it is really best strategy:

To check things out I’ve created table with simple structure:

```bash
mysql> show create table dt2 \G
*************************** 1. row ***************************
       Table: dt2
Create Table: CREATE TABLE `dt2` (
  `grp` int(10) unsigned NOT NULL,
  `slack` varchar(50) DEFAULT NULL
) ENGINE=MyISAM DEFAULT CHARSET=latin1
1 row in set (0.00 sec)
```

Populated it with 75M of rows to reach 4G in size so workload will be IO bound on the box with 2GB of memory.
The was running Fedora Core i686 had 2 Xeon CPUs and 2 drives in RAID0.

I’ve used the following query to perform full table scans, with 3 runs and averaged results. MySQL 5.1.21-beta was used for tests.

```bash
mysql> select count(*) from dt2 where slack like "a%";
+----------+
| count(*) |
+----------+
|  4705992 |
+----------+
1 row in set (51.77 sec)
```

Here are the results I’ve got:

| read_buffer_size | Time (sec) |
| ---------------- | ---------- |
| 8200             | 45.2       |
| 16K              | 44.8       |
| 32K              | 45.6       |
| 64K              | 43.4       |
| 128K             | 43.0       |
| 256K             | 51.9       |
| 512K             | 60.8       |
| 2M               | 65.2       |
| 8M               | 66.8       |
| 32M              | 67.2       |

8200 bytes is the minimum size for read_buffer_size, this is why we start from this value.

As you can see results look really strange. Performance indeed grows by few percent as you increase buffer to 128K but after that instead of improving any further it drops down sharply being 50% slower at 2MB size. After this value it continues to drop slowly all the way to 32M.

Why this is happening ? I have not spent enough time to come up with good explanation. It could be OS has to split large requests into multiple ones submitting them to device which slows things down or it could be something else. But the fact remains – on some platforms for some workloads large read_buffer_sizes may hurt you even on large full table scans. (I [wrote](https://www.percona.com/blog/2006/06/06/are-larger-buffers-always-better/) about some other cases when it hurts a while ago)

Let us do one more test – what if we test out smaller table (which fits in OS cache):

| read_buffer_size | Time (sec) |
| ---------------- | ---------- |
| 8200             | 4.15       |
| 16K              | 4.15       |
| 32K              | 4.12       |
| 64K              | 4.11       |
| 128K             | 4.11       |
| 256K             | 4.12       |
| 512K             | 4.25       |
| 2M               | 4.49       |
| 8M               | 4.54       |
| 32M              | 4.58       |

As you see the difference in percents is smaller with only 10% difference between best and worst numbers but best number still remains the same – 128K and 32M is again the worst value. This means it can’t be request split issues, at least not just that.

**Note:** In this case I’m really curious how much values change on different platforms (OS and Hardware) as well as different file systems as these could all be involved here. Different table structures (ie longer rows) also may affect results, not to mention tables with fragmented rows when IO pattern can be a lot different.

The degree of parallelizm is another important variable which was not considered – small buffers with high concurrency may mean more seeks and so
worse performance, or may be not – something to test as well.

In general it just reconfirms one basic thing – do not just grab someone elses “best configuration” from the web and apply for your application if you’re interested in best performance – experiment with realistic load and realistic data (including fragmentation) to find what works best for you.