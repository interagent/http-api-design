#### Divide Large Responses Across Requests with Ranges

Large responses should be broken across multiple requests using `Range` headers
to specify when more data is available and how to retrieve it. See the
[Heroku Platform API discussion of Ranges](https://devcenter.heroku.com/articles/platform-api-reference#ranges)
for the details of request and response headers, status codes, limits,
ordering, and iteration.
