---
layout: resfulapi
title: symfony
date: 2017-08-10 14:40:08
tags: symfony
---

# resful api bundle for symfony

## Method design:
  - `index`: list of models
  - `view`: return the details of a model
  - `create`: create a new model
  - `update`: update an existing model
  - `delete`: delete an existing model
  - `options`: return the allowed HTTP methods

## Endpoints:

An Endpoint is a URL within your API which points to a specific Resource or a Collection of Resources.
If you were building a fictional API to represent several different Zoo’s, each containing many Animals (with an animal belonging to exactly one Zoo), employees (who can work at multiple zoos) and keeping track of the species of each animal, you might have the following endpoints:

- https://api.example.com/v1/zoos
- https://api.example.com/v1/animals
- https://api.example.com/v1/animal_types
- https://api.example.com/v1/employees

When referring to what each endpoint can do, you’ll want to list valid HTTP Verb and Endpoint combinations. For example, here’s a semi-comprehensive list of actions one can perform with our fictional API. Notice that I’ve preceded each endpoint with the HTTP Verb, as this is the same notation used within an HTTP Request header

- `GET /zoos`: List all Zoos (ID and Name, not too much detail)
- `POST /zoos`: Create a new Zoo
- `GET /zoos/ZID`: Retrieve an entire Zoo object
- `PUT /zoos/ZID`: Update a Zoo (entire object)
- `PATCH /zoos/ZID`: Update a Zoo (partial object)
- `DELETE /zoos/ZID`: Delete a Zoo
- `GET /zoos/ZID/animals`: Retrieve a listing of Animals (ID and Name).
- `GET /animals`: List all Animals (ID and Name).
- `POST /animals`: Create a new Animal
- `GET /animals/AID`: Retrieve an Animal object
- `PUT /animals/AID`: Update an Animal (entire object)
- `PATCH /animals/AID`: Update an Animal (partial object)
- `GET /animal_types`: Retrieve a listing (ID and Name) of all Animal Types
- `GET /animal_types/ATID`: Retrieve an entire Animal Type object
- `GET /employees`: Retrieve an entire list of Employees
- `GET /employees/EID`: Retreive a specific Employee
- `GET /zoos/ZID/employees`: Retrieve a listing of Employees (ID and Name) who work at this Zoo
- `POST /employees`: Create a new Employee
- `POST /zoos/ZID/employees`: Hire an Employee at a specific Zoo
- `DELETE /zoos/ZID/employees/EID`: Fire an Employee from a specific Zoo

## Resourse

`GET: api/v2/users/1`   --->  `UsersResourse:view`  --->  `return json {user}`

`POST: api/v2/users/1`   --->  `UsersResourse:update`  --->  `return jsonEndpoints {user}`

`DELETE: api/v2/users/1`   --->  `UsersResourse:delete`  --->  `return json {bool}`

----

`GET: api/v2/users`   --->  `UsersResourse:index`  --->  `return json {userList}`

`POST: api/v2/users`   --->  `UsersResourse:create`  --->  `return json {user}`

`OPTIONS: api/v2/users` ---> `UsersResourse:options` --->  `return json {options}`

## PathParse


<div id="container"></div>
<link rel="stylesheet" href="https://imsun.github.io/gitment/style/default.css">
<script src="https://imsun.github.io/gitment/dist/gitment.browser.js"></script>
<script>
  var gitment = new Gitment({
    id: '页面 ID', // 可选。默认为 location.href
    owner: 'jonny-bo', // 可以是你的GitHub用户名，也可以是github id
    repo: 'jonny-bo.github.io',
    oauth: {
      client_id: '1c77e0e89cfbef9d889d',
      client_secret: 'b1b0d59ca8a7915cffa91f574f3a4ad720086bbd',
    },
  })
  gitment.render('container')
</script>