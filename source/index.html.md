---
title: ReleaseBits - API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell: curl (json)
  - shell_session: curl (form)

toc_footers:
  - <a href='#'>Sign Up via GitHub for an API Token</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the ReleseBits API! 

The purpose of this API is to easily consume events that occur in any [Continuous Integration](https://en.wikipedia.org/wiki/Continuous_integration) 
and [Delivery](https://en.wikipedia.org/wiki/Continuous_delivery) process in order to pass them on to various external 
APIs: 

  - Grafana
  - Sentry
  - Bugsnag
  - Newrelic
  - Custom Webhooks

It is also planned to soon provide endpoints for Prometheus metrics or build badges based on these events.

# Terminology 

Most software applications will be built from one or more **components** - a monolithic app usually has only very few, whereas in a microservice approach 
there are a lot of individual ones. 

The sucessful creation of a new **version** of given a component is called a **build**. Builds can have errors for 
various reasons thus not creating new versions of the component.

Putting a different version of a component onto a live system is called a **deployment**. Any group of at least one deployment forms a **release**, 
which should enclose a set of meaningful changes for human stakeholders to the software.

# Authentication

> To authorize, use this code:

```shell
# Passing via Authorization header with each request
curl -H "Authorization: some-api-token" \
     "https://api.releasebits.com/"

# Passing via request parameter with each request
curl "https://api.releasebits.com/?access_token=some-api-token"
```

```shell_session
$ curl "https://api.releasebits.com/?access_token=some-api-token"
```

> Make sure to replace `some-api-token` with your API key.

ReleaseBits uses API tokens to allow access to the API. You can register a new API token [here](/account/tokens).

ReleaseBits expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: Bearer some-api-token`

For scripting convenience, you can alternatively specify the token as the `access_token` URL parameter:

`https://api.releasebits.com/?access_token=some-api-token`

<aside class="notice">
You must replace <code>some-api-token</code> with your API token.
</aside>


# Deployments

Denotes that a new version of a given component is has been put onto a live system. This should be recorded individually
for all hosts and stages in order to have good resolution for metrics systems.

We highly recommend setting at least `finished_at` from local system time, in order to keep timestamps in sync with your 
logging systems

## Creating A New Deployment

The only required field to create a new deployment event is the `version` field. However, it is a good idea to submit
as much data as possible in oder to get the most of your 3rd party tooling. 

```shell
curl -XPOST "https://api.releasebits.com/components/component-slug/deployments/" \
  -H "Authorization: Bearer some-api-token" \
  -d ' {
        "version": "v1.2.3",
        "result": "success",
        "stage": "staging",
        "host": "host.example.com",
        "code_name": "Funky Fünke",
        "started_at": "2018-05-27T13:15:30.311Z",
        "finished_at": "2018-05-27T13:15:31.191Z",
        "meta_data": {
          "user_id": "metauser",
          "foo": "bar"
        },
        "commit_sha": "b927e3797fc56fb8bc093d59ee7a624dafddadd2",
        "tag": "tag1.2.3",
        "branch": "release" 
     } '
```

```shell_session
$ curl -XPOST "https://api.releasebits.com/components/component-slug/deployments?access_token=some-api-token&version=v1.2.3&result=success&stage=staging&host=host.example.com&code_name=Funky%20F%C3%BCnke&started_at=2018-05-27T13%3A15%3A30.311Z&finished_at=2018-05-27T13%3A15%3A31.191Z&meta_data[user_id]=metauser&meta_data[foo]=bar&commit_sha=b927e3797fc56fb8bc093d59ee7a624dafddadd2&tag=tag1.2.3&branch=release"
```

> On success the above command returns JSON structured like this along with `201` status:

```json
  { 
    "id": "44b6a0d3-b7a0-4f26-b976-5fce52bfc4b9"
  }
```

### HTTP Request

`POST https://api.releasebits.com/components/{component-slug}/deployments/`

### Query Parameters

Parameter |  | Description
--------- | ------- | -----------
component_slug | (string) | The url slug of the component that is being deployed

### Body Parameters

Parameter |  | Description
--------- | ------- | -----------
version | (string, required)  | The version being deployed, e.g. `v1.2.3` 
result |  (string)  |  The result as a string representation, e.g. `success`
stage |  (string)  | The stage being deployed, e.g.`staging`
host |  (string)  |  The individual host or node being deployed, e.g. `host.example.com`
started_at | (string, iso8601)  |  The UTC date and time the deployment was started, e.g. `2018-04-23T18:25:43.511Z`
finished_at |  (string, iso8601) |  The UTC date and time the deployment was finished, e.g. `2012-04-23T18:25:47.912Z`. Will be set by the system when left empty.
commit_sha | (string) | The full VCS commit sha for the version being deployed, e.g. `b927e3797fc56fb8bc093d59ee7a624dafddadd2`
code_name |  (string)  |  A code name for this release, e.g. `Funky Fünke`
branch |  (string) | The VCS branch being deployed, e.g. `master`
tag | (string) | The VCS tag relating to this deployment, e.g. `v1.2.3`
meta_data | (object) | Arbitrary meta data to be stored along with this deployment

## List Deployments

```shell
curl "https://api.releasebits.com/components/component-slug/deployments"
  -H "Authorization: some-api-token"
```

```shell_session
$ curl "https://api.releasebits.com/components/component-slug/deployments?access_token=some-api-token"
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": "44b6a0d3-b7a0-4f26-b976-5fce52bfc4b9",
    "version": "v1.2.3",
    "result": "success",
    "stage": "staging",
    "host": "host.example.com",
    "code_name": "Funky Fünke",
    "started_at": "2018-05-27T13:15:30.311Z",
    "finished_at": "2018-05-27T13:15:31.191Z",
    "meta_data": {
      "user_id": "metauser",
      "foo": "bar"
    },
    "commit_sha": "b927e3797fc56fb8bc093d59ee7a624dafddadd2",
    "tag": "tag1.2.3",
    "branch": "release" 
  },
  {
     "id": "0ac5875c-e404-47b2-a72d-91760f15e1a2",
    "version": "v1.2.4",
    "result": "success",
    "stage": "staging",
    "host": "host.example.com",
    "code_name": "Funky Fünke",
    "started_at": "2018-05-27T14:11:13.311Z",
    "finished_at": "2018-05-27T14:12:10.118Z",
    "meta_data": {
      "user_id": "metauser",
      "foo": "bar"
    },
    "commit_sha": "b927e3797fc56fb8bc093d59ee7a624dafddadd2",
    "tag": "tag1.2.4",
    "branch": "release" 
  }
]
```

This endpoint retrieves all deployments for a given component.

### HTTP Request

`GET https://api.releasebits.com/components/{component-slug}/deployments`

### Query Parameters

Parameter |  | Description
--------- | ------- | -----------
component_slug | (string) | The url slug of the component that is being deployed
