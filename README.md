# Example StepZen Project with the Storyblok GraphQL API

[Storyblok](https://www.storyblok.com/home) is a headless CMS that includes a visual editor and [various APIs](https://www.storyblok.com/docs/api) to access your content. This example uses StepZen's [`@graphql` directive](https://stepzen.com/docs/connecting-backends/how-to-connect-a-graphql-api) to connect to Storyblok's [GraphQL API](https://www.storyblok.com/docs/graphql-api).

## Setup StepZen Project

`config.yaml` is a file written in YAML to relay information regarding your configuration and keys to StepZen. This is where we set our API token we received from Storyblok. Change `example.config.yaml` to `config.yaml` and include your own token instead of `xxxx`.

```bash
mv example.config.yaml config.yaml
```

```yaml
configurationset:
  - configuration:
      name: storyblok_config
      token: xxxx
```

### Deploy to StepZen

`stepzen start` deploys the code in your current directory to the specified endpoint on StepZen. It also watches the directory for changes and automatically deploys them to the endpoint specified.

```bash
stepzen start
```

### Test the Endpoint

Open a browser window to [localhost:5000/api/stepzen-storyblok](https://localhost:5000/api/stepzen-storyblok) to see the StepZen Schema Explorer. This allows you to test your API by exploring the queries and types available and querying the API running on StepZen. Run the following test query for the `name`, `intro`, and `title` of your post.

```graphql
{
  PostItems {
    items {
      content {
        title
        intro
      }
      name
    }
  }
}
```

![01-stepzen-storyblok-endpoint](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/dfctgok3xzbkhccvenxk.png)

### Project Structure

`stepzen.config.json` gives you the ability to configure different aspects of your StepZen project such as the endpoint name or root directory. Our endpoint is called `api/stepzen-storyblok`. If you do not have a `stpezen.config.json` file then the CLI asks for a name for your endpoint the first time you deploy your project with `stepzen start`.

```json
{
  "endpoint": "api/stepzen-storyblok"
}
```

`index.graphql` is the glue that pulls together all the other `.graphql` files in your schema. Our `index.graphql` file only includes a single schema file called `storyblok.graphql`.

```graphql
schema
  @sdl(
    files: [ "schema/storyblok.graphql" ]
  ) {
  query: Query
}
```

In our schema directory is a file called `storyblok.graphql` for our types and queries. This example shows how to query for just a small subset of all the information available from the Storyblok GraphQL API. If you want to generate a complete schema without needing to code it by hand, you can introspect your endpoint with [`stepzen import graphql`](https://stepzen.com/docs/connecting-backends/graphql-introspection).

```graphql
type PostItems {
  items: [PostItem]
}

type PostItem {
  name: String
  content: Content
}

type Content {
  title: String
  intro: String
}

type Query {
  PostItems: PostItems
    @graphql(
      endpoint: "https://gapi.storyblok.com/v1/api"
      headers: [
        { name:"Token" value:"$token" }
      ]
      configuration: "storyblok_config"
    )
}
```

Our types include a `PostItem` with a `name` for the post and a `Content` type for the `title` and `intro` of the post. The `PostItems` type returns an array of those `PostItem` objects. Our query includes the `endpoint` and sets the `headers` for our token. `configuration` includes our token from `config.yaml`.

## Create a Blog Post on Storyblok

You can sign up for Storyblok at the [following link](https://app.storyblok.com/#!/signup). After signing up, you are asked whether you want to use a demo or create a new space. A space is a content repository for keeping all the content related to one project. Each space has its own components, datasources, assets, environments, domains, collaborators, and permissions.

![02-choose-your-own-adventure](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wnbcfftvuwtkfpvj62fs.png)

Choose to create a new space instead of playing with a demo so we can build up our content from scratch.

### Storyblok Dashboard

After creating a new space you are taken to your Storyblok dashboard. You can manage your content here. Create a new folder and give your folder a name and slug, then create a new entry and use the Post content blueprint to create a blogpost.

![03-create-your-first-entry](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0qsb83sjey4ijpog0quk.png)

The post editor gives you the ability to create your blog post and include images, an intro, and an author.

![04-post-editor-with-post-data](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/cnr0mbx5g7295lglzcma.png)

The JSON response looks like the following:

```json
{
  "_uid": "ec395043-549c-43d5-a389-6e3334e1f35a",
  "component": "Post",
  "title": "My Super Awesome Post",
  "image": "//a.storyblok.com/f/122199/1920x1080/823e44a555/gorillas-shark-explosion-high-five-wallpaper.jpeg",
  "intro": "This is the intro to my super awesome post. It isn't a very long intro.",
  "long_text": {
    "type": "doc",
    "content": [
      {
        "type": "paragraph",
        "content": [
          {
            "type": "text",
            "text": "This is the long text of my super awesome post. It is slightly longer than the intro."
          }
        ]
      }
    ]
  },
  "author": ""
}
```

### Connect to the Storyblok GraphQL API

You can find your API keys listed under the Settings tab. Use this token to access Storyblok's GraphiQL editor.

![05-settings-api-keys](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5apxi6yqm01rcjktl10z.png)

Storyblok includes a [GraphiQL editor](https://gapi-browser.storyblok.com/) similar to StepZen to explore your data. Include `?token=YOUR_TOKEN` after the URL to access your specific content. Open the Explorer tab to see the different queries that are available. The queries include the content itself as well as information about your content and space.

![06-graphiql-explorer](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2p7py4pjlflb2q1jfkn7.png)

Send the following test query returning the `title`, `intro`, and `long_text` of the blog posts to make sure you are connected.

```graphql
{
  PostItems {
    items {
      content {
        title
        intro
        long_text
      }
    }
  }
}
```

![07-graphiql-query](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3q56dms0a4nty4fk5qqz.png)

## Next Steps

Curious to learn more about Storyblok and how it can improve your StepZen project? You can see Facundo Giuliani and Anthony Campolo walk through creating content in Storyblok and connecting to StepZen in [How to mix data from Storyblok CMS with your own project using StepZen](https://www.youtube.com/watch?v=gDxYEUIzRMQ).

You can sign up for StepZen at the [following link](https://stepzen.com/signup). If you've got more questions or just want to hang out you can also join us [on our Discord](https://discord.gg/T2st3mCp).