# Errors


The API uses the following error codes:


Error Code | Meaning
---------- | -------
400 | Bad Request -- Your request is malformed or you are missing a required parameter
401 | Unauthorized -- Your session has expired or you are not providing the correct session/authentication
404 | Not Found -- The resource you are attempting to access does not exist
405 | Method Not Allowed -- The API supports only GET for most resources.  Login supports POST and Logout supports DELETE.
500 | Internal Server Error -- An error has occurred with the API.  If you continue to have issues please open an [Issue](https://github.com/vkapi/idyia/issues).
503 | Service Unavailable -- Servers are under heavy load.  Please try again later.
