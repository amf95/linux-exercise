

> Testing is what is going to determine if your resources are enough.

> A good ratio between PHP-FPM workers and nunber of concurrent users is `1:1.5`.

> During testing 10,000 Apache workers, 2,000 PHP-FPM workers, redis 2500 allowed connections and database 2,500 allowed connections could handle 10,000 concurrent requests to the home page, CPU was old intel-xeon that is equivilat to `Intel Core i7-7700` -> 95% utilization, and RAM usage was about `25-30`GB.