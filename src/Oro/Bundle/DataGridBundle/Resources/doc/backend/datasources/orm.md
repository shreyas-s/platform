ORM datasource
===============

Overview
--------

This datasource provide adapter to allow access data from doctrine orm using doctrine query builder.
You can configure query using `query` param under source tree. This query will be converted via [YamlConverter](../../../../Datasource/Orm/QueryConverter/YamlConverter.php) to doctrine `QueryBuilder` object.

Example
-------

``` yaml
datagrids:
    DATAGRID_NAME_HERE:
        source:
            type: orm
            query:
                select:
                    - g.id
                    - g.label
                from:
                    - { table: OroContactBundle:Group, alias: g }
```

Modification of a query configuration from PHP code
---------------------------------------------------

Sometime it is required to modify a query configuration from PHP code, for example from datagrid [extensions](../extensions.md) or [listeners](../datagrid.md#extendability). This can be done using [OrmQueryConfiguration](../../../../Datasource/Orm/OrmQueryConfiguration.php) class. To get an instance of this class use `getOrmQuery` method of [DatagridConfiguration](../../../../Datagrid/Common/DatagridConfiguration.php). For example:

```php
$query = $config->getOrmQuery();
$rootAlias = $query->getRootAlias();
$query->addSelect($rootAlias . '.myField');
```

In additional to a query modification methods, the [OrmQueryConfiguration](../../../../Datasource/Orm/OrmQueryConfiguration.php) contains several useful methods like:

- `getRootAlias()` - Returns the FIRST root alias of the query.
- `getRootEntity($entityClassResolver = null, $lookAtExtendedEntityClassName = false)` - Returns the FIRST root entity of the query.
- `findRootAlias($entityClass, $entityClassResolver = null)` - Tries to find the root alias for the given entity.
- `getJoinAlias($join, $conditionType = null, $condition = null)` - Returns an alias for the given join. If the query does not contain the specified join, its alias will be generated automatically. This might be helpful if you need to get an alias to extended association that will be joined later.

Please investigate this class to find out all other features.

Query hints
-----------

The following example shows how [query hints](https://doctrine-orm.readthedocs.org/en/latest/reference/dql-doctrine-query-language.html#query-hints) can be set:

``` yaml
datagrids:
    DATAGRID_NAME_HERE:
        source:
            type: orm
            query:
                select:
                    - partial g.{id, label}
                from:
                    - { table: OroContactBundle:Group, alias: g }
            hints:
                - HINT_FORCE_PARTIAL_LOAD
```

If you need to set hint's value you can use the following syntax:

``` yaml
datagrids:
    DATAGRID_NAME_HERE:
        source:
            type: orm
            query:
                select:
                    - c
                from:
                    - { table: %oro_contact.entity.class%, alias: c }
                join:
                    left:
                        - { join: c.addresses, alias: address, conditionType: WITH, condition: 'address.primary = true' }
                        - { join: address.country, alias: country }
                        - { join: address.region, alias: region }
            hints:
                - { name: HINT_CUSTOM_OUTPUT_WALKER, value: %oro_translation.translation_walker.class% }
```

Please pay attention that ORM datasource uses [Query Hint Resolver](./../../../../../EntityBundle/Resources/doc/query_hint_resolver.md) service to handle hints. If you create own query walker and wish to use it in a grid, just register it in the Query Hint Resolver. For example the hint `HINT_TRANSLATABLE` is registered as an alias for the translation walker and as result the following configurations are equal:

``` yaml
            hints:
                - { name: HINT_CUSTOM_OUTPUT_WALKER, value: %oro_translation.translation_walker.class% }

            hints:
                - HINT_TRANSLATABLE
```
