# Drupal Services Overview

This document provides examples and resources for creating resources via the [Services][services] module.

To expose new resources directly via Services, implement [hook_services_resources()][hsr]. You should define either new operations, actions, or targeted actions as per hook documentation.

Note that when you define new CRUD operations, services will automatically expose new URLs for the new resource according to the following pattern:

* Create: POST /[endpoint_path]/[resource] + body data
* Retrieve: GET /[endpoint_path]/[resource]/[resource_id]
* Update: PUT /[endpoint_path]/[resource]/[resource_id] + body data
* Delete: DELETE /[endpoint_path]/[resource]/[resource_id]

These default URL patterns necessitate a default, corresponding argument structure. For example:

* CREATE operations expect argument ‘source’ to be defined as ‘data’
* UPDATE operations expect at least two arguments, one with 'source' defined as 'path', and one with 'source' defined as 'data'
* etc.


## See Also

* [Working with REST Server](https://drupal.org/node/783254)
* [Posting data to a services endpoint](http://drupal.org/node/1334758)

[services]: http://drupal.org/project/services
[hsr]: http://drupalcontrib.org/api/drupal/contributions!services!services.services.api.php/function/hook_services_resources/7
