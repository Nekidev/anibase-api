# AniBase API v1
AniBase - Anime and Manga (version 1.0.0)

Base url: [https://api.anibase.co/v1](https://api.anibase.co/v1)

## What you should know
Welcome to the AniBase API! We're glad you are considering using our API for your amazing project 😍. Before you start diving in, there are some things that you should know.

### How the server is built
We (actually I'm a single dev, you get the point) have built our servers using Python 3.10 and the [Django framework](https://www.djangoproject.com/). This is why many things, such as the filtering and other things relate mainly to the Django ORM and other things it provides. For example, a filter looks like this: `filter[titleCanonical.icontains]=seishun+buta`. Here, we are using the Django ORM lookup expressions to specify the match type of the filter. This can be very useful to add some extra customization to your app. We still provide a more complex and simple at a time search filter to make things easier if you don't really require to customize your queries that much. Most of the times, the `filter[search]` filter will work better with matches than filters with lookup expressions. For example, when searching for `Rent-A-Girlfriend`, a request containing `filter[search]=rent+a+gi` will return `Rent-A-Girlfriend` as a result, while `filter[titleCanonical.icontains]=rent+a+gi` will not, as `icontains` searches for an exact substring in the `titleCanonical` field and the field has hyphens, which the query does not. Instead, the `filter[search]` query parameter will return the anime as a result because the query  made is more complex. 

## The JSON:API Specification
AniBase's API (sort of) follows the JSON:API Spec, which you can find at [https://jsonapi.org](https://jsonapi.org). The JSON:API schema is very extense and we do our best to follow it, but there are some (small) modifications you will notice.

### Specifying request headers
The API will not respond to any request which does not have the following headers.
```http
Accept: application/vnd.api+json
Content-Type: application/vnd.api+json
```
The `Accept` header is required in every request. However, the `Content-Type` header is only required if the request has a JSON body. This is because the server responses will (almost) always follow the `JSON:API` spec, but not always you will need to send a body in the request.

### Pagination
Pagination follows the JSON:API pagination schema. Here is an example of a paginated response.
```json
{
  "data": [
    {
      "type": "anime",
      "id": "1",
      "attributes": {
        "titles": {},
        "titleCanonical": ""
      }
    }
  ],
  "links": {
    "prev": "url",
    "next": "url",
    "self": "url",
    "first": "url",
    "last": "url"
  },
  "meta": {
    "pagination": {
      "count": 115,
      "limit": 50,
      "offset": 0
    },
    "language": "en"
  }
}
```
You can specify the pages with the `page[limit]` and `page[offset]` query parameters. The `page[limit]` is the amount of objects returned per page, and the `page[offset]` is the start index of the page. For example, a page with `page[limit]=10` and `page[offset]=20` will return items from 21 to 30. Notice that for demonstration, brackets are shown without encoding, but you should make requests using `%5B` (`[`) and `%5D` (`]`). The same applies for any query parameter.

Paginated responses have 3 properties in their root object: `data`, `links` and `meta`. The `data` property will return an array with all objects for the page, the `links` property has an object with links for pagination, and the `meta` property has non-standard properties, which usually are a `pagination` property with some extra details about the pagination, and a `language` property, which is the current language of the request. For more information about response's language, refer to the [language](#language) section.

### Filtering and searching
You can query the data returned in paginated response by using filters. These are query parameters that follow a `filter[name.lookup]=value` format. The regex would be `filter\[([a-zA-Z|_]+).{0,1}([a-zA-Z|_]+)?\]` if it's easier for you.

#### Filter `name`
The filter name specifies the field you are filtering. For example, if you want to filter the query to only return animes with the genre `comedy`, then you will have to add a query parameter like this: `filter[genres]=comedy`.

#### Filter lookup expression
As we previously metioned, we use the Django ORM default lookup expressions for filtering. Some of this are:
- `icontains`: Contains a substring (case insensitive)
- `contains`: Contains a substring (case sensitive)
- `iexact`: Exact match (case insensitive)
- `exact`: Exact match (case sensitive)

There are more lookup expressions (not all lookup expressions are allowed for all fields, and some fields cannot have lookup expressions) which you can find at the Django documentation.
