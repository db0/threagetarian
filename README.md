# Threativore

A Fediverse/Threadiverse bot to fight against spam and help with moderation

The name is a portmanteau of Threats + Vore. I.e. this is a tool that eats threats.

# Setup

## Clone the repo

_This is easiest to do within your Lemmy folder you have your docker compose already in_

`git clone https://github.com/db0/threativore.git`

Then move into the new `threativore` directory.

## Option 1 - Run directly via python

* `python -m pip install -r requirements.txt`
* Copy `.env_template` into `.env` and modify its contents according to the comments
* `python run.py`

## Option 2 - Run with docker

You can easily run threativore alongside your existing docker containers for Lemmy.  If you used the `docker-compose` method of installing Lemmy this is even easier.

### Docker Compose

_Note: Remember to modify the yaml as needed, for example granting access to the Lemmy networks_

#### Using Volumes (and sqlite)

Modify your existing `docker-compose.yml` and add threativore as another container.  An example may look like
```yaml
  automod:
    build:
      context: ./threativore # Note this is the same directory you cloned above
      dockerfile: Dockerfile
    environment:
      - LEMMY_DOMAIN=<<yourlemmydomain.foo >>
      - LEMMY_USERNAME=<<Your Bot's Username (created within Lemmy)>>
      - LEMMY_PASSWORD=<<Your Bot's Password (created within Lemmy)>>
      - USE_SQLITE=1
      - THREATIVORE_ADMIN_URL=<<Full URL to your own admin profile, such as lemmy.foo/u/yourusername>>
    restart: always
    volumes:
      - /path/to/volume:./volumes
    ports:
      - 8000:8080
```

#### Using postgres (separate container)
```yaml
  automod:
    build:
      context: ./threativore # Note this is the same directory you cloned above
      dockerfile: Dockerfile
    environment:
      - LEMMY_DOMAIN=<<yourlemmydomain.foo >>
      - LEMMY_USERNAME=<<Your Bot's Username (created within Lemmy)>>
      - LEMMY_PASSWORD=<<Your Bot's Password (created within Lemmy)>>
      - USE_SQLITE=0
      - THREATIVORE_ADMIN_URL=<<Full URL to your own admin profile, such as lemmy.foo/u/yourusername>>
      - POSTGRES_USER=<<postgres username>>
      - POSTGRES_URL=<<postgres url>>
      - POSTGRES_PASS=<<postgres password>>
```

# Use

This bot control is currently via PMs. Soon I'll be added a REST API interface as well however.

This means that user and filter management happens through PMing it specifically formatted texts.

## Filters

The bot uses regexp filters to catch easily detectable spam comments as defined by the people in charge.

### Add

Send the PM in the following markdown format

```
threativore add comment filter: `trial period`
reason: `Spam comment`
action: `REMOVE`
description: `Known spam string`
```

You can use whitespace instead of linebreaks if you want, but you **have** to wrap the variables in `backticks`

If the filter was added succesfully, you will receive a reply back informing you of the changes.

The various fields are:

* filter: the regexp to use when searching posts
* reason: the reason to use for the modlog
* action: what type of action to take when this filter is triggered. The options are
   * REMOVE: Remove the comment/post
   * BAN7: Remove and Ban for 7 days
   * BAN30: Remove and Ban for 30 days
   * PERMABAN: Remove and Ban forever
   * REPORT: Automatically report the comment/post
* description: internal description for the filter to be used by admin collaboration.

Also important, the initial format has to be somewhat consistent
* The PM needs to start with `threativore`
* the `comment` in "threativore add **comment** filter" is a keyword and **shouldn't** be wrapped in `backticks`. The available options are
   * comment: a filter for comments
   * report: a filter for reports
   * url: a filter for post urls
   * username: a filter for commenter/poster usernames

### Remove

Similar format like [#Add](#add) but use "remove" instead and you only need the filter type (i.e. "comment") and the regex. I.e.

```markdown
threativore remove comment filter: `trial period`
```
### Modify

Similar format like [#Add](#add) but use "modify" instead and you only need the filter type (i.e. "comment"), the regex, and then the updated regex, or any other changing fields. Keep in mind that the filter_type here (e.g. "comment") will changed on the filter if it's different

 I.e.


```
threativore modify report filter: `trial period`
new filter: `trial run`
reason: `Spam comment`
action: `REMOVE`
description: `Known spam string`
```

### Show

Use this format to show all filters matching the provided regex. The bot will report with all the details of each filter which has this string in their regexp

```
threativore show comment filter: `period`
```

### List

Use this format to show all filters of a specific type

```
threativore list comment filters
```

## Users

Only specific users have access to define the configuration of this bot

There's three user roles

#### ADMIN

The bot has one admin, which is defined in the `.env`. The admin role cannot be removed or added any other way, and it's the only one which can add new moderators

#### MODERATOR

Any user can be a moderator (even from other instances). Moderators have access to see and modify all filters and to add trusted users

#### TRUSTED

Any user can be trusted (even from other instances). Trusted users' reports get preferrential treatment and they can also be given some command permissions

### Add Roles

The bot allows you to set new roles via PM. To do so, you need to use this format

```
threativore add user: [@db0@lemmy.dbzer0.com](https://lemmy.dbzer0.com/u/db0)
role: `moderator`
```

The user **has** to be in url format, i.e. `https://lemmy.dbzer0.com/u/db0`, not `@db0@lemmy.dbzer0.com`. You can also use the lemmy autocomplete for user accounts however. The markdown hyperlink will also work.

### Remove roles

Works the same way, but use `remove` instead of `add`. E.g.

```
threativore remove user: https://lemmy.dbzer0.com/u/db0
```
