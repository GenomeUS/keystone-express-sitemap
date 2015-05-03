# keystone-express-sitemap

> Dynamic sitemap generation for applications built on KeystoneJS

## Introduction

keystone-sitemap allows you to set up your [KeystoneJS](http://keystonejs.com/) app to return a dynamic sitemap.xml file that is aware of all database-driven content within your site.

For example, this set of routes in routes/index.js:
```
app.get('/', routes.views.index);
app.get('/recipe/:_id', routes.views.recipe);
app.get('/order', routes.views.order);
```

will be parsed into:
```
<?xml version="1.0" encoding="UTF-8"?>
	<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
		<url>
			<loc>http://mysite.com/</loc>
		</url>
		<url>
			<loc>http://mysite.com/recipe/1214</loc>
			<lastmod>2015-03-03</lastmod>
		</url>
		<url>
			<loc>http://mysite.com/recipe/8183</loc>
			<lastmod>2014-04-01</lastmod>
		</url>
		<url>
			<loc>http://mysite.com/recipe/6685</loc>
			<lastmod>2015-04-03</lastmod>
		</url>
		<url>
			<loc>http://mysite.com/order</loc>
		</url>
	</urlset>
```

Note that the keystone admin application routes (/keystone/*) will not be part of the sitemap. An option to choose whether or not to include these routes may be included in a later version. If you're impatient, feel free to fork me and set it up yourself.

## Installation

1. Install the package and save it with the rest of your site dependencies
	```
	npm install keystone-express-sitemap --save
	```

2. Add the sitemap.xml route to the rest of your site routes

	**routes/index.js**
	```
	var keystone = require('keystone'),
		sitemap = require('../keystone-sitemap');

		// other middleware/dependencies go here

		exports = module.exports = function(app) {
			app.get('/sitemap.xml', function(req, res) {
				sitemap.create(keystone, req, res);
			});

			// other application routes go here
		}
	```

3. Go to sitemap.xml on your domain and watch the SEO magic fairy dust sprinkle itself magically over your site.

### Usage notes
* The sitemap generator works with dynamic routes declared in 2 formats:

	`/[list name]/:[dynamic parameter]`
	`/[any custom route string]/:[list name]`

	For example, if you have an app with a `Recipe` list, your sitemap would include all of the items in the `Recipe` database collection if your app used any of these routes:

	* `/recipe/:_id`
	* `/recipe/:name`
	* `/noms/:recipe`
	* `/om/nom/nom/:recipe`

* The sitemap generator is currently designed to only handle routes with one dynamic parameter. It will not behave as expected if it is used to try and parse routes such as `/recipe/:om/:nom`.

* In order for dynamic routes to be parsed properly, the name of the list tied to that content must be in one part of the declared route, either as the directory or the dynamic parameter. If you have an app with a `Recipe` list, the following routes will **not** work:

	* `/rec-ipe/:_id`
	* `/noms/:_id`
