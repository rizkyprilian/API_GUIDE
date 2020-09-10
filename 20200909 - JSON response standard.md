# Fetching Resource

## Top Level

Standard JSON response format

```
 {
   "data": [{}], // documents primary data (resource object)
   "errors": [{}], // array of error object
   "meta": {}, // non-standard meta information

   // optional
   "links": {}, // a links object related to the primary data.
   "included": [{}] // array of resource object
 }
```

> - **data and errors tidak boleh muncul bersamaan**
> - **Kalau data kosong, include jg harus kosong**


## Error Object

Dalam suatu kondisi, server akan return error atau tetap melanjutkan proses walaupun ada kondisi error (bisa lebih dari satu error -- dalam bentuk array).

Error object ini membantu app frontend untuk cek hasil response API

```
if ('errors' in response) {
  // handle error
}
```

> **Apabila terjadi error, jangan gunakan HTTP Status Code 200**

```
  "errors" = [{
    "id": 212313, // [optional] event id. mempermudah tracing logs di server
    "code": 312, // [required] custom error code, perlu dibuat dokumentasi
    "status": 500, // [required] HTTP Status Code
    "title": "InvalidParameter", // [required] Human readable error string
    "detail": "kode_kec must be an integer", // [optional] penjelasan error yang lebih panjang (optional)
    "source": { // [required]
      "parameter" : "kode_kec" // referensi object yang menyebabkan error
    },
    "meta": { // [optional] non-standard meta version
      "version": "1.0"
    },
    "links": {
      "about": "https://gitlab.com/docs" // [optional] panduan dokumentasi misalnya
    }]
  }
```

## Resource Object

Minimal dalam resource object ada id dan type. untuk field yang lain yang terkait dengan object tersebut, bisa dimasukkan ke object "attributes"

"relationships" menjelaskan hubungan relation dengan API resource yang lain (optional)

```
{
  "type": "articles",
  "id": "1",
  "attributes": {
    "title": "Rails is Omakase"
  },
  "relationships": {
    "author": {
      "links": {
        "self": "/articles/1/relationships/author",
        "related": "/articles/1/author"
      },
      "data": { "type": "people", "id": "9" }
    }
  }
}
```

Apabila resource yang diminta dalam bentuk collection atau array, maka "data" juga harus dalam bentuk array atau collection.

```
// collection
GET /app/items

"data": [
  {
    "id": 1,
    "type": "items"
  },
  {
    "id": 2,
    "type": "items"
  },
  ...
]

// single resource

GET /app/items/1

"data": {
  "id": 1,
  "type": "items"
}

```

### Included object

Dalam suatu kondisi, untuk menghemat jumlah API Call, aplikasi frontend bisa meminta langsung object-object yang terkait dengan resource utama yang diminta.


```
// include query
GET /articles/1/relationships/comments?include=comments.author


// response

"data": {
  "id": "1",
  "type": "posts",
},
"included": [{
    "type": "people",
    "id": "9",
    "attributes": {
      "first-name": "Dan",
      "last-name": "Gebhardt",
      "twitter": "dgeb"
    },
    "links": {
      "self": "http://example.com/people/9"
    }
  }, {
    "type": "comments",
    "id": "5",
    "attributes": {
      "body": "First!"
    },
    "relationships": {
      "author": {
        "data": { "type": "people", "id": "2" }
      }
    },
    "links": {
      "self": "http://example.com/comments/5"
    }
  }, {
    "type": "comments",
    "id": "12",
    "attributes": {
      "body": "I like XML better"
    },
    "relationships": {
      "author": {
        "data": { "type": "people", "id": "9" }
      }
    },
    "links": {
      "self": "http://example.com/comments/12"
    }
  }]
```


## Links Object

Minimal ada self untuk penanda current resource URI

```
"links": {
  "self": "http://example.com/articles?page[number]=3&page[size]=1", // current resource URI/ page
}
```

Apabila mau ditambahkan meta object di tiap link object

```
"links": {
  "related": {
    "href": "http://example.com/articles/1/comments",
    "meta": {
      "count": 10
    }
  }
}
```

### Pagination

```
GET /app/items?limit=10
```

Untuk response server dalam jumlah banyak, bisa dilakukan pagination

- first: API URI halaman pertama
- prev: API URI halaman sebelumnya (apabila ada di halaman pertama, value nya null)
- next: API URI halaman setelahnya (apabila ada di halaman terakhir, value nya null)
- last: API URI halaman terakhir (apabila jumlah halaman satu, value nya sama dengan "first")

```
"links": {
    "self": "http://example.com/articles?page[number]=3&page[size]=1", // current resource URI/ page
    "first": "http://example.com/articles?page[number]=1&page[size]=1",
    "prev": "http://example.com/articles?page[number]=2&page[size]=1",
    "next": null, // misal halaman terakhir (berguna untuk app frontend yang infinite scrolling)
    "last": "http://example.com/articles?page[number]=13&page[size]=1"
  }
```

Sementara untuk informasi total pages, bisa dimasukkan ke meta object

```
  "meta": {
    "totalPages": 13,
    "numPerPage": 10
  }
```

### Filter

Aplikasi frontend juga bisa meminta filter resource

```
// contoh filtering
GET /app/items?finished_at=ge:15:30&finished_at=lt:16:00

// response

"meta" : {
  "filter": {
    "finished_at": {
      "ge": "15:30"
    },
  }"
}
```

## Sort

Untuk keperluan sorting, aplikasi frontend dapat override default sort method. penambahan - didepan object key untuk sort descending

```
// contoh sort
GET /app/items?sort=-created_at,age

// response
"meta": {
  "sort": "-created_at,age"
}
```

## Meta Object

The value of each meta member MUST be an object.

```
{
  "meta": {
    "copyright": "Copyright 2015 Example Corp.",
    "authors": [
      "John Doe",
      "Jane Doe"
    ]
  },
  "data": {
    // ...
  }
}
```

----

# Aturan Penamaan Key

## Allowed Characters

Karakter paling aman untuk digunakan sebagai nama key:

    U+0061 to U+007A, “a-z”
    U+0041 to U+005A, “A-Z”
    U+0030 to U+0039, “0-9”
    U+0080 and above (non-ASCII Unicode characters; not recommended, not URL safe)

Untuk memisahkan kata, bisa menggunakan - atau _ :

    U+002D HYPHEN-MINUS, “-“
    U+005F LOW LINE, “_”

## Reserved Characters

Karakter dibawah ini tidak boleh digunakan untuk penamaan key:

    U+002B PLUS SIGN, “+” (used for ordering)
    U+002C COMMA, “,” (used as a separator between relationship paths)
    U+002E PERIOD, “.” (used as a separator within relationship paths)
    U+005B LEFT SQUARE BRACKET, “[” (used in sparse fieldsets)
    U+005D RIGHT SQUARE BRACKET, “]” (used in sparse fieldsets)
    U+0021 EXCLAMATION MARK, “!”
    U+0022 QUOTATION MARK, ‘”’
    U+0023 NUMBER SIGN, “#”
    U+0024 DOLLAR SIGN, “$”
    U+0025 PERCENT SIGN, “%”
    U+0026 AMPERSAND, “&”
    U+0027 APOSTROPHE, “’”
    U+0028 LEFT PARENTHESIS, “(“
    U+0029 RIGHT PARENTHESIS, “)”
    U+002A ASTERISK, “*”
    U+002F SOLIDUS, “/”
    U+003A COLON, “:”
    U+003B SEMICOLON, “;”
    U+003C LESS-THAN SIGN, “<”
    U+003D EQUALS SIGN, “=”
    U+003E GREATER-THAN SIGN, “>”
    U+003F QUESTION MARK, “?”
    U+0040 COMMERCIAL AT, “@”
    U+005C REVERSE SOLIDUS, “\”
    U+005E CIRCUMFLEX ACCENT, “^”
    U+0060 GRAVE ACCENT, “`”
    U+007B LEFT CURLY BRACKET, “{“
    U+007C VERTICAL LINE, “|”
    U+007D RIGHT CURLY BRACKET, “}”
    U+007E TILDE, “~”
    U+007F DELETE
    U+0000 to U+001F (C0 Controls)

## Todo

- Standar format POST, UPDATE request dari client dan response dari server


## Referensi

- [JSON API](https://jsonapi.org/format/#status)
- [Pagination, Sort,Filtering](https://specs.openstack.org/openstack/api-wg/guidelines/pagination_filter_sort.html)