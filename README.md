# AniBase API v1
AniBase - Anime and Manga (version 1.0.0)

Base url: [https://api.anibase.co/v1](https://api.anibase.co/v1)

## What you should know
Welcome to the AniBase API! We're happy to know you are considering using our API for your amazing project 😍. Before you start diving in, there are some things that you should know.

### How the server is built
We (actually I'm a single dev, you get the point) have built our servers using Python 3.10 and the [Django framework](https://www.djangoproject.com/). This is why many things, such as the filtering and other things relate mainly to the Django ORM.

## The JSON:API Specification
AniBase's API (sort of) follows the JSON:API Spec, which you can find at [https://jsonapi.org](https://jsonapi.org). The JSON:API schema is very extense and we do our best to follow it, but there are some (small) modifications you will notice.

### Specifying request headers
The API will not respond to any request that does not have the following headers.
```http
Accept: application/vnd.api+json
Content-Type: application/vnd.api+json
```
The `Accept` header is required in every request. However, the `Content-Type` header is only required if the request has a JSON body. This is because the server responses will (almost) always follow the `JSON:API` spec, but not always you will need to send a body in the request.

### Pagination
Pagination follows the JSON:API pagination schema. Here is an example of a paginated response.
```javascript
{
    "data": [
        // [Insert response objects]
    ],
    "links": {
        "prev": Url,
        "next": Url,
        "self": Url,
        "first": Url,
        "last": Url
    },
    "meta": {
        "pagination": {
            "count": Integer,
            "limit": Integer,
            "offset": Integer
        },
        "language": String
    }
}
```
You can specify the pages with the `page[limit]` and `page[offset]` query parameters. The `page[limit]` is the amount of objects returned per page, and the `page[offset]` is the start index of the page. For example, a page with `page[limit]=10` and `page[offset]=20` will return items from 21 to 30. Notice that for demonstration, brackets are shown without encoding, but you should make requests using `%5B` (`[`) and `%5D` (`]`). The same applies for any query parameter.

Paginated responses have 3 properties in their root object: `data`, `links` and `meta`. The `data` property will return an array with all objects for the page, the `links` property has an object with links for pagination, and the `meta` property has non-standard properties, which usually are a `pagination` property with some extra details about the pagination, and a `language` property, which is the current language of the request. For more information about response's language, refer to the [language](#language) section.

### Resource objects
Resource objects also follow a specific format. You can read all the details at [jsonapi.org/format/](https://jsonapi.org/format/#document-resource-objects). A resource object follows the following format:
```javascript
{
    "type": "object-type",
    "id": "resource-id",
    "attributes": {},
    "relationships": {},
    "links": {}
}
```

From the JSON:API specification:
> The `type` member is used to describe resource objects that share common attributes and relationships.
> The `id` member is not required when the resource object originates at the client and represents a new resource to be created on the server.

`id` and `type` properties will always be strings, although in most cases the `id` will contain only numbers.

The `attributes` property contains all the resource object's attributes. For example, an `anime` resource which contains a `title` and a `genres` property will return something like this:
```javascript
{
    "type": "anime",
    "id": "12.5", // Actual IDs have no decimals
    "attributes": {
        "title": "Cute lil' cat",
        "genres": [
            "Comedy",
            "Slice of life"
        ]
    },
    "links": {
        "self": {
            "href": "https://api.anibase.co/anime/12.5",
            "meta": {
                "website": "https://anibase.co/anime/12.5"
            }
        }
    },
    "relationships": {
        "characters": {
            "data": [
                { "type": "character", "id": "1" },
                { "type": "character", "id": "2" }
            ]
        }
    }
}
```

The `links` object returns additional information about links (yes, you've guessed right). You can find more information about the `links` object at [jsonapi.org/format/](https://jsonapi.org/format/#document-links).

The `relationships` property and the `links` property are optional.

#### Relationships
A resource object's relationships are returned in the `relationships` property. This is like the `attributes` property, but only for relationships. You can find more information about `relationships` at [jsonapi.org/format/](https://jsonapi.org/format/#document-resource-object-relationships).

**WARNING: We do not support the `include` query parameter and neither we send an error when you use it in the requests, as specified in the JSON:API spec.**

### Filtering and searching
You can query the data returned in paginated response by using filters. These are query parameters that follow a `filter[name.lookup]=value` format. The regex would be `filter\[([a-zA-Z|_]+).?([a-zA-Z|_]+)?\]`.

#### Filter `name`
The filter name specifies the field you are filtering. For example, if you want to filter the query to only return animes with the genre `comedy`, then you will have to add a query parameter like this: `filter[genres]=comedy`.

#### Filter lookup expression
As we previously metioned, we use the Django ORM default lookup expressions for filtering. Some of this are:
- `icontains`: Contains a substring (case insensitive)
- `contains`: Contains a substring (case sensitive)
- `iexact`: Exact match (case insensitive)
- `exact`: Exact match (case sensitive)
- `isnull`: If the field is null or not

There are more lookup expressions (not all lookup expressions are allowed for all fields, and some fields cannot have lookup expressions) which you can find at the Django documentation.

#### The search filter
In some cases, there is an extra filter parameter, the `filter[search]` parameter. This parameter makes a more complex query than the other filters, so we strongly recommend using it if allowed. This parameter has no lookup expressions, so doing `filter[search.iexact]` will make the filter invalid and won't work.

#### Filter value types
Depending on the filter, values are formatted in different ways. This usually depends on the type of the field and the lookup expression being used.

| Name     | Format                            |
|----------|-----------------------------------|
| `string` | Plain text                        |
| `number` | A number, with or without decimals|
| `choice` | An option from a list of choices  |
| `list`   | A `,` delimited list of values.   |
| `boolean`| `true`, `false` or `null`         |

`lists` can contain multiple `choice` values. For example, when filtering animes by genre, you can do `filter[genres]=romance,comedy` to return animes with both genres.

#### Sorting
Paginated responses can be sorted. To achieve this you can use the `sort` query parameter. The `sort` parameter requires a `list`-type value of fields. For example, `?sort=rating` will sort objects by the rating field, leaving the lowest at the top and the highest at the bottom. You can invert the sorting by adding a `-` prefix to each item of the list. For example, `?sort=-rating,titleCanonical` will sort the results leaving the highest rating objects at the top and ordered from a-z when 2 or more have the same rating value.

This is an example of how items would be ordered with `?sort=-rating,titleCanonical`
```javascript
[
    { "titleCanonical": "Awesome lil' cats", "rating": 9.5 },
    { "titleCanonical": "Beautiful lil' cats", "rating": 9.5},
    { "titleCanonical": "Amazing lil'cats", "rating": 8 }
]
```

## Language
The API currently supports two languages, English and Spanish. By default, API responses are in English. When requests are authenticated, the default language is the user's preferred language.

You can specify the language by using the `language` query parameter.
- `en`: English
- `es`: Spanish (Español)

**WARNING: If something is missing translation, it will return `null` instead of the default English value. This means that although x field cannot return a null value, it will if it is not translated. If your app supports translation, consider that anything can return null at any time.**
