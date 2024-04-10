# laminas-filter
**Fix the DATE error when generating any type of report in Magento 2**

When trying to generate reports in Magento nothing happens. An error similar to this appears in the log files:

```
#121 {main} {"exception":"[object] (Zend_Db_Statement_Exception(code: 0): SQLSTATE[HY000]: General error: 1525 Incorrect DATE value: '3/31/24', query was: SELECT sum(orders_count) AS orders_count, sum(orders_invoiced) AS orders_invoiced, sum(invoiced) AS invoiced, sum(invoiced_captured) AS invoiced_captured, sum(invoiced_not_captured) AS invoiced_not_capturedFROMsales_invoiced_aggregated_order WHERE (period >= '3/1/24') AND (period <= '3/31/24') AND (store_id IN(1)) at /vendor/magento/framework/DB/Statement/Pdo/Mysql.php:109) [previous exception] [object] (PDOException(code: HY000): SQLSTATE[HY000]: General error: 1525 Incorrect DATE value: '3/31/24' at /vendor/magento/framework/DB/Statement/Pdo/Mysql.php:90)"} []
```


The source of the problem is updated laminas component - https://github.com/laminas/laminas-filter/.

https://github.com/laminas/laminas-filter/blob/298b53c8cdcdb6c1b567fb81435642bef11ef348/src/FilterChain.php#L90

`if (is_callable($callback)) {
    $this->attach($callback, $priority);
}`

**This patch applies a temporary fix:**

1 - create this file called: reverter-laminas-filter.patch
```
--- a/laminas-filter/src/FilterChain.php
+++ b/laminas-filter/src/FilterChain.php
@@ -87,7 +87,7 @@
                     foreach ($value as $spec) {
                         $callback = $spec['callback'] ?? false;
                         $priority = $spec['priority'] ?? static::DEFAULT_PRIORITY;
-                        if (is_callable($callback)) {
+                        if ($callback) {
                             $this->attach($callback, $priority);
                         }
                     }
```

2 - put it into your patches folder, in this case /patches/ at the root directory
3 - Make sure you have cweagans installed to apply the patch.
(these are SSH commands)

Check package: `composer show cweagans/*`

Install the package: `composer require cweagans/composer-patches`

4 - Make a copy of your composer.json and save it, just in case.
5 - At the end of your composer.json add the following code for the patch to be applied. Note that you need to specify the correct patch file path:

        "extra": { "magento-force": "override", "patches": { "laminas/laminas-filter": { "Resolve o erro 1525 Incorrect DATE value nos relatórios": "patches/reverter-laminas-filter.patch" } } }

5 - Apply Patches: With the patches properly defined in your composer.json, you will need to update your dependencies for Composer to apply the patches. This can be done with the following command:

`composer install`

Or, if you already installed your dependencies and just added new patches:
`composer update`

If everything goes well, the patch will be applied and you will follow it on the console. If necessary, review the procedures above.
Remembering that you must run the basic Magento commands of upgrade, cleaning and deploy.

I hope I contributed.
