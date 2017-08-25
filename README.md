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
		sitemap = require('keystone-express-sitemap');

		// other middleware/dependencies go here

		exports = module.exports = function(app) {
			app.get('/sitemap.xml', function(req, res) {
				sitemap.create(keystone, req, res);
			});

			// other application routes go here
		}
	```

3. Go to sitemap.xml on your domain and watch the SEO magic fairy dust sprinkle itself magically over your site.

## Additional options
The sitemap create function accepts an optional options object parameter, which can be used to include additional information about your route structure.

### Hide route
To ignore a route completely from the sitemap, declare the route string in the `ignore` array in the same way as you declared the route string within your site routes.

```
app.get('/sitemap.xml', function(req, res) {
	sitemap.create(keystone, req, res, {
		ignore: ['/secret/:id']
	});
});
```

If you want to ignore multiple routes with a common pattern, you can pass the regular expression that matches that pattern.

```
app.get('/sitemap.xml', function(req, res) {
	sitemap.create(keystone, req, res, {
		ignore: ['^\/api.*$']
	});
});
```

### Hidden page filter
Some Keystone models may include a boolean to show/hide or publish/unpublish a specific piece of content tied to a model. You can use the `filters` option to declare the model name and the filter condition that should be applied to the routes that use that model to exclude any pages that are not shown/published from the sitemap file.

The `filters` object may have a parameter for each model in your application. If no parameter is declared for a model name, then the sitemap assumes that all records in the collection for that model should be included in the sitemap.

If the custom route filtering parameter is declared, then it must specify the function that should be used for filtering records of that route's model. The function should accept the model object as a parameter, and should evaluate to `true` for pages that should be included in the sitemap.

```
app.get('/sitemap.xml', function(req, res) {
	sitemap.create(keystone, req, res, {
		filters: {
			'Recipe': function(recipe) {
				return recipe.hide !== true;
			}
		}
	});
});
```

### Custom list field as a URL
You can provide a custom key for sitemap generator to retreive a value from. In the next example a custom field named `slug` will be used instead of default fallback:

```
app.get('/sitemap.xml', function(req, res) {
	sitemap.create(keystone, req, res, {
		customId: {
			'Page': 'slug'
		}
	});
});
```

This option may allow you to define custom fields for routes, that look like this one — `/:page`.


## Usage notes
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
