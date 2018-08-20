## PHP: Indirect modification of overloaded element of %s has no effect

### Prepare php-src

```sh
git clone https://github.com/php/php-src.git
```

### Search

```sh
> ag -G '.c$' 'modification of overloaded element'
Zend/zend_execute.c
1859:				zend_error(E_NOTICE, "Indirect modification of overloaded element of %s has no effect", ZSTR_VAL(ce->name));
1868:						zend_error(E_NOTICE, "Indirect modification of overloaded element of %s has no effect", ZSTR_VAL(ce->name));
```

### Zend/zend_execute.c

```sh
> git blame

> git log 7aa7627172c11979ec45c2db85f99182812ee59d

Use ZSTR_ API to access zend_string elements (this is just renaming without semantick changes)
```

```c
1855 >...>...>...if (UNEXPECTED(retval == &EG(uninitialized_zval))) {
1856 >...>...>...>...zend_class_entry *ce = Z_OBJCE_P(container);
1857
1858 >...>...>...>...ZVAL_NULL(result);
1859 >...>...>...>...zend_error(E_NOTICE, "Indirect modification of overloaded element of %s has no effect", ZSTR_VAL(ce->name));
1860 >...>...>...} else if (EXPECTED(retval && Z_TYPE_P(retval) != IS_UNDEF)) {
1861 >...>...>...>...if (!Z_ISREF_P(retval)) {
1862 >...>...>...>...>...if (result != retval) {
1863 >...>...>...>...>...>...ZVAL_COPY(result, retval);
1864 >...>...>...>...>...>...retval = result;
1865 >...>...>...>...>...}
1866 >...>...>...>...>...if (Z_TYPE_P(retval) != IS_OBJECT) {
1867 >...>...>...>...>...>...zend_class_entry *ce = Z_OBJCE_P(container);
1868 >...>...>...>...>...>...zend_error(E_NOTICE, "Indirect modification of overloaded element of %s has no effect", ZSTR_VAL(ce->name));
1869 >...>...>...>...>...}
```

### See

+ https://stackoverflow.com/questions/20053269/indirect-modification-of-overloaded-element-of-splfixedarray-has-no-effect
+ http://blog.ishinao.net/2007/05/28/45/
