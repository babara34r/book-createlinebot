# ปัญหาที่พบ



```
failed to create symbolic link '/app/.heroku/php/php': File exists

```

he issue was occuring because i had added buildpack of php multiple times .Once i removed all of them and added only once it worked like a charm.



ref : https://github.com/heroku/heroku-buildpack-php/issues/181











