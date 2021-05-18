## What's this?

I had to migrate an unmaintained and old Wordpress subdirectory multisite from shared hosting to a VPS. I am proud of my Google-fu but couldn't find much working or helpful information about getting a subdirectory setup working with Trellis. Hopefully this collection of things that worked for me™ will save you some time and frustration.

## What you need to change
All occurrences of /blog/ should be changed to match your path_current_site.

## nginx-includes
Once vagrant up'ed and wp db imported to your dev server, you'll find that wp-admin either doesn't work at all or causes an endless redirect. Neither the default rewrites nor the [roots/multisite-url-fixer](https://github.com/roots/multisite-url-fixer)  nor [felixarntz/multisite-fixes](https://github.com/felixarntz/multisite-fixes) were helpful for my subdirectory setup.

https://github.com/roots/multisite-url-fixer/issues/9#issue-817246045 discovered some working rewrite rules. I have put them in nginx-includes. The second rule is necessary for wp-login etc to redirect to example.com/blog/wp-... instead of example.com/wp-...

## Deploys

### tmp_multisite_constants.php error
I've commented out the two lines that cause the error: https://github.com/roots/trellis/issues/1025#issuecomment-471291803


###  WordPress database error Table 'example.wp_blogs' doesn't exist

Your first deploy may fail near the [very end](https://github.com/roots/trellis/issues/482#issue-132127282). Try this:
 
```bash
$ ssh web@example.com 
$ cd /srv/www/blog/releases/2021...
$ wp core multisite-install --allow-root --title="Test" --admin_user="admin" --admin_password="admin" --admin_email="admin@example.com"
```

Now, cross your fingers and deploy again.

## Importing your database

```bash
$ ssh web@example.com
$ cd /srv/www/blog/current/
$ wp db import db.sql
$ wp search-replace --network example.test example.com
```

- Don't forget the \-\-network
- Run the search-replace twice for good luck


## Bonus: Fixing upload URLs in a network created pre-WP 3.5
If your multisite was originally created years ago, it may have images stored in blogs.dir/01... . I couldn't figure out how to get the Apache rewrite for that working with nginx. Thankfully, a way to make your network use uploads/sites/01/... is detailed here:

https://anchor.host/removing-legacy-ms-files-php-from-multisite/

The script is quite cool, but I ended up doing the steps manually:

```bash
$ trellis db open --app=tableplus development blog
```

- In wp_sitemeta, set ms_files_rewriting to 0. Create it if it doesn't exist.
- In wp_options for each of your sites, set upload_path and upload_url_path to empty.
- Copy your uploads folders from blogs.dir/01... to uploads/sites/01...

## To-do

- fix rewrite rules so example.com/blog/wp-admin/ also works without the trailing slash
- Work on incorporating these fixes into the Trellis docs

## Thanks

- The Roots team for making Trellis
- Everyone who worked hard to get their own subdirectory multisites working in Trellis, and
- [michaelw85 ](https://github.com/michaelw85) for help pointing me in the right direction.

