---
layout: post
title: Code Reading - Learn ❤️ GraphQL
---

## What is [GraphQL](http://graphql.org/)?

- "Graph Query Language"
- Alternative to REST pattern
- Making your data queryable through a single endpoint

## Why use GraphQL?

- More declarative
- More consumer focused
- Simplified endpoints
- Fewer requests
  - With REST, request count grows w/ more data you need
  - With GraphQL, only request as much data as you need => more efficient

## [In use in the wild](http://graphql.org/users/)

- Github API
- Shopify

## Stuff to Think About

- Due to HTML limitations, super long queries get sent via POST request
- [Mutations](http://graphql.org/learn/queries/)

## Demo

1. Spin up new rails app: `rails new graphql-demo`
2. Add [`graphql-ruby` gem](https://github.com/rmosolgo/graphql-ruby) to Gemfile
3. Run `rails generate graphql:install`
  - gives you `graphql` directory with sub-directories for mutations, types, new controller, new route `graphql-rails`
4. Lookit `/grpahiql` loader dev tool
5. Lookit `/app/graphql/graphql_demo_schema.rb`
6. Lots of demo boilerplate-y examples generated for us
7. Add more field queries to `Types::QueryType`
  ```ruby
  field :course, Types::CourseType do
    description "This will return a single course"
    argument :id, !types.Int # bang means required
    resolve->(obj, args, ctx){
      Course.find(args["id"])
    }
  end

  field :customString, types.String do
    description "This will return a custom string"
    resolve->(obj, args, ctx){
      "custom string"
    }
  end
  ```
8. Define types in `/graphql/types` dir
  ```ruby
  Types::CourseType = GraphQL::ObjectType.define do
    name "Course"
    field :name, types.String
    field :description, types.String
  end

  Types::StudentType = GraphQL::ObjectType.define do
    name "Student"
    field :first_name, types.String
    field :last_name, types.String
  end
  ```
9. Make the request
  ```
  {
    course(id: 1) {
      name
      description
    }
    student(id: 2) {
      first_name
    }
  }
  ```
10. Note: doesn't handle 404s very gracefully. Tries to parse error message as JSON, so you end up w/ strange error responses
11. How to do associations
  ```ruby
  # type
  Types::CourseType = GraphQL::ObjectType.define do
    name "Course"
    field :name, types.String
    field :description, types.String
    field :students, Types::StudentType.to_list_type
      resolve -> (obj, args, ctx) {
        obj.students
      }
  end

  # request
  {
    course(id: 1) {
      name
      students {
        first_name
        last_name
      }
    }
  }
  ```
