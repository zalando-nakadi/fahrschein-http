# Fahrschein HTTP API and Implementation

[![Build Status](https://travis-ci.org/zalando-incubator/fahrschein-http.svg?branch=master)](https://travis-ci.org/zalando-incubator/fahrschein-http)
[![Release](https://img.shields.io/github/release/zalando-incubator/fahrschein-http.svg)](https://github.com/zalando-incubator/fahrschein-http/releases)
[![Maven Central](https://img.shields.io/maven-central/v/org.zalando/fahrschein-http.svg)](https://maven-badges.herokuapp.com/maven-central/org.zalando/fahrschein-http)

A fork of the spring [`ClientHttpRequestFactory`](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/http/client/ClientHttpRequestFactory.html)
api and implementations without dependency on the remainder of the spring framework.
The main usecase is for the [fahrschein nakadi client](https://github.com/zalando-incubator/fahrschein).

## Changes compared to the spring implementation

 - The `ClientHttpResponse#close` methods do not try to consume remaining data from the stream, instead the connection is aborted
   (see [SPR-14040](https://jira.spring.io/browse/SPR-14040) and [SPR-14882](https://jira.spring.io/browse/SPR-14882)).

## Usage

### Sending a request body serialized with [Jackson](https://github.com/FasterXML/jackson) and deserializing the response

```java
    final URI uri = URI.create(...);
    final ObjectMapper objectMapper = new ObjectMapper();

    // RequestData and ResponseData can be arbitrary java objects which can be serialized and deserialized by jackson
    final RequestData requestData = ...

    final SimpleClientHttpRequestFactory clientHttpRequestFactory = new SimpleClientHttpRequestFactory();
    requestFactoryDelegate.setConnectTimeout(500);
    requestFactoryDelegate.setReadTimeout(60 * 1000);

    final ClientHttpRequest request = clientHttpRequestFactory.createRequest(uri, HttpMethod.POST);

    request.getHeaders().setContentType(MediaType.APPLICATION_JSON);

    try (final OutputStream os = request.getBody()) {
        objectMapper.writeValue(os, requestData);
    }

    try (final ClientHttpResponse response = request.execute()) {
        try (final InputStream is = response.getBody()) {
            final ResponseData responseData = objectMapper.readValue(is, ResponseData.class);
            ...
        }
    }
```

### Using the [Apache HttpComponents](https://hc.apache.org/) implementation

```java

final RequestConfig config = RequestConfig.custom().setSocketTimeout(readTimeout)
                                                   .setConnectTimeout(connectTimeout)
                                                   .setConnectionRequestTimeout(requestTimeout)
                                                   .build();

final CloseableHttpClient httpClient = HttpClients.custom()
                                                  .setConnectionTimeToLive(readTimeout, TimeUnit.MILLISECONDS)
                                                  .disableAutomaticRetries()
                                                  .setDefaultRequestConfig(config)
                                                  .disableRedirectHandling()
                                                  .setMaxConnTotal(8)
                                                  .setMaxConnPerRoute(2)
                                                  .build();

final ClientHttpRequestFactory clientHttpRequestFactory = new HttpComponentsClientHttpRequestFactory(httpClient);
```

## Getting help

If you have questions, concerns, bug reports, etc, please file an issue in this repository's issue tracker.

## Getting involved

To contribute, simply make a pull request and add a brief description (1-2 sentences) of your addition or change.
