# Postman Configuration - REST API Reference
#
To test the REST API using [Postman](https://www.getpostman.com/), requests need to include the headers required for [authentication](../REST/authentication/hmac.md). Here's how to configure Postman for testing the REST API, generating the authentication headers automatically:

1. Create a new [request](https://learning.getpostman.com/docs/postman/sending_api_requests/requests/)

2. Add the `signRequest` function from the [JavaScript authentication sample](../REST/authentication/hmac.md#JavaScript) to the [pre-request script](https://learning.getpostman.com/docs/postman/scripts/pre_request_scripts/) for the request

3. Add the following code to the end of the pre-request script and update the access key as indicated by the TODO comment. Also review whether the code update the URL for signing is needed (based on your Postman settings as indicated by the TODO comment).

    ```js
    // TODO: Replace the following placeholders with your access key
    var credential = "<Credential>"; // Id
    var secret = "<Secret>"; // Value

    var pathWithQuery = pm.request.url.getPathWithQuery();

    // TODO: The following block of code updates the URL for signing based on Postman's default URL
    // processing behavior, which encodes additional characters such as '*'.
    //
    // If you have enabled Postman's new URL processing behavior 'Use next generation URL processing',
    // remove this block of code to avoid authentication failures from a mismatched signing URL.
    {
        var updatedQuery = [];
        pm.request.url.query.each(query => {
            value = query.value.replace(/[!'()*\\]/g, c => '%' + c.charCodeAt(0).toString(16).toUpperCase());
            updatedQuery.push({"key": query.key, "value": value});
        });

        pathWithQuery = pm.request.url.getPath() + "?" + updatedQuery.map(query => query.key + '=' + query.value).join('&');
    }

    var isBodyEmpty = pm.request.body === null || pm.request.body === undefined || pm.request.body.isEmpty();

    var headers = signRequest(
        pm.request.url.getHost(),
        pm.request.method,
        pathWithQuery,
        isBodyEmpty ? undefined : pm.request.body.toString(),
        credential,
        secret);

    // Add headers to the request
    headers.forEach(header => {
        pm.request.headers.upsert({key: header.name, value: header.value});
    })
    ```

4. Send the request
