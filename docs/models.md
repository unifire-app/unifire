## unifire.model.user

aka `user`

- `id` (any)
- `email` (string)
- `status` (int, enum)
  - `0` - active, OK
  - `1` - disabled
- `inviter` (any, user.id)
- `auth` (object)
  - `method` (string)
  - `cred` (table)
- `modified` (datetime)
- `created` (datetime)

**methods:**

- `find_by_id(id): user`
- `find_by_email(email): user`
- `find_all_by_project(project.id): user[]`
- `find_all_by_company(company.id): user[]`
- `find_all_by_auth_method(method): user[]`
- `insert(user)`
- `update(user)`
- `delete(user)`

**sorting:**

- by `email`

**unique fields**

- `id`
- `email`

## unifire.model.company

aka `company`

- `id` (any)
- `name` (string) company host name. 
- `title` (string)
- `members` (collection). Document:
  - `user_id` (any, user.id)
  - `access_level` (int, enum)
    - `0` — member
    - `1` — master
    - `2` — maintainer
    - `3` — owner
  - `inviter` (any, user.id)
  - `corrector` (any, user.id) 
  - `modified` (datetime)
  - `created` (datetime)
- `creator` (any, user.id) 
- `corrector` (any, user.id) 
- `modified` (datetime)
- `created` (datetime)

**methods:**

- `find_by_id(id): company`
- `find_by_name(name): company`
- `find_by_member(user.id): company[]`
- `insert(company)`
- `update(company)`
- `delete(company)`

**sorting:**

- by `name`

**unique fields**

- `id`
- `name`

## unifire.model.project

aka `project`

- `id` (any)
- `company_id` (any, company.id)
- `name` (string) project name, domain (`company.name` + `project.subdomain` = `project.name`) 
- `subdomain` (string)
- `vars` (collection, `name` as key). Document:
  - `name` (string)
  - `value` (string)
  - `policy` (int, enum)
    - `0` - public. read all users and use everywhere (queries, templates).
    - `1` - protected. 
    - `2` - private. read/edit only owner, only queries can access, stored encrypted.
- `policy` (int, enum)
  - `0` - public. All without authorization have access to the pages
  - `1` - protected. Access to pages is only through authorization.
  - `2` - private
- `members` (collection):
  - `user_id` (any, user.id)
  - `access_level` (int, enum)
    - `0` — member. Read/write templates, create/edit/delete pages
    - `1` — master. member + read/create/edit/delete queries
    - `2` — maintainer. master + edit project info, read/edit protected variables.
    - `3` — owner. maintainer + read/edit private project variables, delete project, rename project
  - `inviter` (any, user.id)
  - `corrector` (any, user.id) 
  - `modified` (datetime)
  - `created` (datetime)
- `recaptcha` (object)
  - `secret` (string)
  - `min_score_level` (int)
- `auth` (object)
  - `method` (string)
  - `config` (table)
- `author` (any, user.id) 
- `corrector` (any, user.id) 
- `modified` (datetime)
- `created` (datetime)

**methods:**

- `find_by_id(id): project`
- `find_by_name(name): project`
- `find_by_member(user.id): project[]` list user's project, ordered by `name`
- `find_by_modified(modified): project[]`
- `insert(project)`
- `update(project)`
- `delete(project)`

**sorting:**

- by `name`

**unique fields**

- `id`
- `name`
- `company_id`, `subdomain`

## unifire.model.template

aka `template`

- `id` (any)
- `project_id` (any, project.id) project link
- `name` (string) the template name
- `body` (string) aspect template code
- `parent` (string) parent template name
- `blocks` (collection, `name` as key). Defined blocks.
  - `name` (string) block name
  - `description` (string) block description
  - `body` aspect block code
  - `vars` (collection, `name` as key). List of global variables used in the block.
    - `name` (string) variable name
    - `role` (int, enum) how is the variable used
      - `0` — plain. variable used as is.
      - `1` — iterator. used as iterator
      - `2` — hash. used as hash
      - `3` - mixed. iterator and hash
    - `keys` (string) what keys were used
  - `refs` (collection, `name` as key). list of references to other templates.
    - `name` (string) template name
    - `tag` (string) tag name that refers to another template
- `macros` (collection, `name` as key). Defined macros.
  - `name` (string) macros name
  - `description` (string) macros description
  - `args` (collection). List of arguments
    - `name` (string) argument name
    - `default` (string) argument default value
- `content` (object). Template information outside of blocks.
  - `vars` (collection, `name` as key). List of global variables used in the template.
    - `name` (string)  variable name
    - `role` (int, enum) how is the variable used
      - `0` — plain. variable used as is.
      - `1` — iterator. used as iterator
      - `2` — hash. used as hash
      - `3` - mixed. iterator and hash
    - `keys` (string) what keys were used
  - `refs` (collection, `name` as key). list of references to other templates.
    - `name` (string) template name
    - `tag` (string) tag name that refers to another template
- `total` (object) blocks + content 
  - `vars` (collection). Document:
    - `name` (string)
    - `role` (int, enum)
      - `0` — plain. variable used as is.
      - `1` — iterator. used as iterator
      - `2` — hash. used as hash
      - `3` - mixed. iterator and hash
    - `keys` (string)
  - `refs` (collection). Document:
    - `name` (string)
    - `tag` (string)
- `author` (any, user.id) 
- `corrector` (any, user.id) 
- `modified` (datetime)
- `created` (datetime)

**sorting:**

- by `name`

**methods:**

with `project` context

- `find_by_id(id): template`
- `find_by_name(name): template`
- `find_many(): template[]` ordered by `name`
- `find_by_variable(variable_name): function` iterator with `model`, `block_name`  
- `find_by_refs(template_name): template[]`
- `insert(template)`
- `update(template)`
- `delete(template)`
- `truncate()`

**unique fields**

- `id`
- `name`

## unifire.model.query

aka `query`

**fields:**

- `id` (any)
- `project_id` (any, project.id) project link
- `page_id` (any, page.id) page specific query (`0` if global)
- `name` (string)
- `description` (string)
- `api` (string)
- `method` (string)
- `args` (collection)
  - `name` (string)
  - `type` (int, enum)
    - `0` - undefined
    - `1` - string
    - `2` - reference to the project variable
    - `3` - reference to the value from the request
    - `4` - reference to the value from the API
  - `value` (string) value or reference name
- `cache`:
  - `mode` (int, enum)
    - `0` - disabled
    - `1` - runtime cache (cache on 1)
    - `2` - longtime cache. uses `cache.ttl` 
  - `ttl` (int)
- `author` (any, user.id) 
- `corrector` (any, user.id) 
- `modified` (datetime)
- `created` (datetime)

**methods:**

with `project` context

- `find_by_id(id): query`
- `find_by_name(name): query`
- `find_many_global(): query[]` list of all global queries, ordered by `name`
- `find_many_for_pages(): query[]` list of all page specific queries, ordered by `name`
- `find_many_by_page(page.id, [with_globals]): query[]` list of all queries for page and global queries (if `with_globals` is true), ordered by `name`
- `insert(query)`
- `update(query)`
- `delete(query)`
- `truncate()`

**unique fields**

- `id`
- `project_id`, `name`

## unifire.model.page 

aka `page`

**fields:**

- id (any)
- `project_id` (any, project.id) project link
- `name` (string) the URL with placeholders
- `is_pattern` (boolean)
- `action` (collection, `method` as key)
  - `method` (string) request HTTP method (GET/HEAD, POST, PUT, PATCH, DELETE)
  - `type` (int, enum)
    - `0` — render the template using (`page.action.template` and `page.action.layout`) 
    - `1` — evaluate specific script
    - `2` - evaluate specific script and render the template
  - `template` (string) the template name  
  - `layout` (string) parent of the template (see [template inheritance](https://aspect.unifire.app/syntax.html#template-inheritance))
  - `script` (string) the lua script 
  - `author` (any, user.id) 
  - `corrector` (any, user.id) 
  - `modified` (datetime)
  - `created` (datetime)
- `query` (collection)
  - see model `unifire.model.query`
- `author` (any, user.id) 
- `corrector` (any, user.id) 
- `modified` (datetime)
- `created` (datetime)

**methods:**

with `project` context

- `find_by_id(id:any): page`
- `find_by_name(URL): page`
- `find_by_pattern(URL): page`
- `find_many_by_template(template_name:string): page[]`
- `find_many_by_layout(layout_name:string): page[]`
- `find_many_by_script(script_name:string): page[]`
- `insert(page)`
- `update(page)`
- `delete(page)`
- `truncate()`

**unique fields**

- `id`
- `project_id`, `name`, `action.method`

# Loading models

For example GET to http://sample.unifire.app/github/unifire-app/aspect/merges:

```http
GET /github/unifire-app/aspect/merges
Host: sample.unifire.app
```

Runs code like

```lua
local project = repo.project:find_by_name("sample.unifire.app")
local company = repo.company:find_by_id(project.company_id)
local page = repo.page:find_by_name("/github/unifire-app/aspect/merges") 
if not page then
    -- page pattern /github/:organization/:project/merges
    page = repo.page:find_by_pattern("github", "unifire-app", "aspect", "merges", "")
end
local queries = repo.query:find_many_global()
local template = repo.template:find_by_name(page.action["GET"].template)

queries = table.merge(queries, page.queries)

local data = api.hub:load(queries)

aspect:display(template.name, data)
```

**Note.** Search by pattern (DQL)

page in storage:

```yaml
## ...
name: "/github/:organization/:project/merges"
pattern:
  prefix: github # first segment of the URL
  hook: merges # second (static) segment of the URL
## ...
```

DQL query will be like

```sql
SELECT * FROM page WHERE page.pattern.prefix = 'github' AND page.pattern.hook IN ("unifire-app", "aspect", "merges", "")
```

Then try match URL for each selected rows. 
For example `/github/:organization/:project/merges` converts to `/github/(%s+)/(%s+)/merges` lua string pattern.