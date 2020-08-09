# Web Development Workshop 
Welcome to the Web Development Workshop. We will be using Gatsby, React, and Netlify CMS to create a personal portfolio and blog. 

The gatsby starter can be found at https://github.com/devsuoa/devs-gatsby-starter

# Prerequisites
* Install Git to your computer: https://git-scm.com/book/en/v2/Getting-Started-Installing-Git
* Create a Github profile here: https://github.com/

Please create an empty repository using github

Install a text editing software such as VS Code / Notepad ++ / Sublime text etc
You can change the default text editor for git to the text editor of your choice: https://help.github.com/en/github/using-git/associating-text-editors-with-git

- [Visual Studio Code](https://code.visualstudio.com/): A text editor which we will be writing our code in.
- [Node.js](https://nodejs.org/en/): Used to run our javascript code.

## Windows

A video of the setup process has been recorded for you if you get stuck. https://youtu.be/Yv-Se-KbFa8

### Visual Studio Code

- Download the windows installer from https://code.visualstudio.com/
- Run the installer once downloaded
- Follow the installation steps

## Node.js

- Download the windows LTS installer from https://nodejs.org/en/download/
- Run the installer once downloaded
- Follow the installation steps
- Open the windows command prompt (search for cmd)
- Verify that node is installed by running `node --version`. The node version (e.g. v12.16.0) should be returned.

## MacOS

A video of the setup process has been recorded for you if you get stuck. https://www.youtube.com/watch?v=Ntv5XS4NBfU

We will use homebrew to install all of the software for mac. Install homebrew by pasting the following two commands into the terminal (spotlight and search for terminal).

```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
```
export PATH="/usr/local/bin:$PATH"
```

### Visual Studio Code

- Run the following command in the terminal

```
brew cask install visual-studio-code
```

### Node.js

- Run the following command in the terminal

```
brew install node
```

- Verify that node is installed by running `node --version`. The node version (e.g. v12.16.0) should be returned.

## This is the end of the pre-requisites! The steps below are for the practical, and you do NOT have to complete these right now (we'll go through them in the workshop)

## Part 2: Creating a blog

### Uploading to Github

Run on terminal
```
  git add -A
  git commit -m "Made personal website"
```

Then make a repo on github

Run on terminal
```
git remote add origin <url>
git push -u origin master
```

### Plugins

Gatsby plugins are Node.js packages that implement Gatsby APIs. These plugins are defined in the `gatsby-config.js` file. 

Plugins are either listed as 
```
  { 
    resolve: <plugin_name>
    options: {
      option1: ...
    }
  }
```

**OR** if no options are needed, then the plugin name will suffice

Looking into the `gatsby-config.js` file, we find:

1. `gatsby-source-filesystem`: This plugin will load all the files specified in the path i.e. `/blog` option into File nodes. 
2. `gatsby-transformer-remark`: This plugin will parse markdown type File Nodes into MarkdownRemark nodes. They are easily queryable in this form.
3. `gatsby-plugin-netlify-cms`: This plugin will automatically create the cms side of the website. 

Note: these plugins were already installed when we ran `gatsby new`. You can find more plugins to add at https://www.gatsbyjs.org/plugins/


### Netlify CMS Configuration

Configuration for netlify cms is found in the `config.yml` file at `static/admin`. 

Looking into the `config.yml` file we find: 

```yml
backend:
  name: github
  repo: <github_name>/<repo_name>
```
The backend option specifies how to access the content for your site, including authentication. i.e. where should netlify cms look for/post blog posts. In this case, netlify cms will be uploading our blog posts to the github repo.

```yml
media_folder: static/assets
public_folder: /assets
```
The media_folder option specifies the folder path where uploaded files should be saved. e.g. images that are uploaded will be saved in the `static/assets` folder.

The public_folder option specifies the folder path where the files uploaded by the media library will be accessed, relative to the base of the built site. e.g. gatsby will move images to the `/assets` folder when the website is built.

Now, we must configure the actual blog part. We use the `collections` option to specify different content types. e.g. a blog or events. 

```yml
collections:
  - name: blog
    label: Blog
    folder: blog
    create: true
    fields:
      - { name: date, label: Date, widget: datetime }
      - { name: title, label: Title }
      - { name: author, label: Author }
      - { name: body, label: Body, widget: markdown }
```

Looking at the fields: 

- `name` is a unique identifier for a collection.
- `label` is the string displayed on the CMS page for this collection
- `folder` specifies what folder to save our blog posts to. In this case `/blog`
- `create` specifies whether users are allowed to create new blog posts
- `fields` include the format of the blog post. In this case, we have a date, title, author and body. Note that if `widget` is not specified, the type defaults to string. Also note that `body` contains the actual markdown of the blog post. The rest is just frontmatter (i.e. metadata of the post)

More documentation of configuration options are found here: https://www.netlifycms.org/docs/configuration-options/

#### **`/static/admin/config.yml`**
``` yml
backend:
  name: github
  repo: <github_name>/<repo_name>

media_folder: static/assets
public_folder: /assets

collections:
  - name: blog
    label: Blog
    folder: blog
    create: true
    fields:
      - { name: date, label: Date, widget: datetime }
      - { name: title, label: Title }
      - { name: author, label: Author }
      - { name: body, label: Body, widget: markdown }
```

### Running the CMS

Run on terminal
`gatsby develop` or `npm run start`

Go to http://localhost:8000/admin/ and login with github.

Click on **New Blog** and fill out the fields and click **Publish**.

This will publish the created blog post as a markdown file to github. To retrieve this blog post, run `git pull` on terminal.

So far, we have a way of creating markdown posts in our repository via an interface. Now we focus on displaying our blog posts. 

### Querying Blog Posts with GraphQL. 

This page will contain all of our blog posts. We will do this by first finding all the markdown posts in `/blog`, parsing them and mapping them. 

Thankfully, majority of the work is done by the gatsby plugins we installed earlier (gatsby-transformer-remark). All we have to do is write a GraphQL query to retrieve the parsed markdown. 

GraphQL is a query language used to retrieve data from API's. 

Gatsby allows GraphQL to be used to retrieve any data related to the website e.g. outputted from plugins.  

As explained before, the `gatsby-transformer-remark` plugin creates a MarkdownRemark Node for each of the markdown files in the `/blog` folder. 

A GraphQL query looks like this: 
```
{
  site {
    siteMetadata {
      title
    }
  }
}
```
The important thing is that the response will be an javascript object containing keys corresponding to schema you queried for.  

In this case, the response may look like this : 

```
{
  "site": {
    "siteMetadata": {
      "title": "A Gatsby site!"
    }
  }
}
```

We can construct GraphQL queries easily by using the GraphQL client that gatsby provides when we run `gatsby develop`. 

Go to http://localhost:8000/___graphql

Notice that clicking on the types on the left side, constructs a query for you. 

To query for our markdown blog posts, we construct the following GraphQL query: 

```
allMarkdownRemark {
  nodes {
    frontmatter {
      author
      date
      title
    }
    html
    id
  }
}
```

Executing the query may return the following response: 

```
{
  "data": {
    "allMarkdownRemark": {
      "nodes": [
        {
          "frontmatter": {
            "author": "Alan",
            "date": "2020-08-07T23:39:01.559Z",
            "title": "My First Blog Post"
          },
          "html": "<p>This is a blog post</p>",
          "id": "f144c37b-ac1f-5343-8e28-cb5a99421e2c"
        }
      ]
    }
  },
  "extensions": {}
}
```

### Creating a Blog page

Gatsby's integration with GraphQL allows working with GraphQL extremely easy. Gatsby will execute exported graphql queries and inject them into the react component as a prop.

First create a new file `blog.js` in `src/pages`

In this file we import the following: 
```js
import React from "react"
import { Link, graphql } from "gatsby"
```

and type our query in the file: 

```js
export const pageQuery = graphql` {
    allMarkdownRemark {
      nodes {
        frontmatter {
          author
          date(formatString: "MMMM DD, YYYY")
          title
          path
        }
        excerpt
        id
      }
    }
  }
`
```

and our blog component as follows: Note the { data } prop that is passed in. 

```js
export default function Blog({ data }) {
    return (
    <section>
        {data.allMarkdownRemark.nodes
            .map(post => {
                return (
                    <article key={post.id}>
                        <h1>
                            <Link to={post.frontmatter.path}>{post.frontmatter.title}</Link>
                        </h1>
                        <h2>{post.frontmatter.date}</h2>
                        <p>{post.excerpt}</p>
                    </article>
                )
            })
        }
    </section>
    )
}
```

Clicking on the title, leads to a 404 page. This is because we havent created a page for each blog post. 

### Creating a page for each blog post

How do we create a page for each blog post? Clearly, we cant manually create a new js file in `/pages` whenever we get a new blog post.

What we really need is a way to programatically create pages when gatsby is building the website. We can achieve what we want by using the `gatsby-node.js file`. 

One of the functions passed as an action to `exports.createPages()` is `createPage()`. This will allow us to create pages at build time! 

The createPage() function takes 3 parameters we care about :

`path`: The path of the page that is being created i.e. localhost:8000/**path**

`component`: A react component that will display our blog post i.e. a template

`context`: Information passed into the react component. We could potentially pass in the entire blog post info right here, but we are opting not to do this and instead just pass the post id. 


#### Lets create a blog post component !
First create a new js file `blog-post.js` at `src/templates`

Next, write the component assuming a blog post id is passed into the graphQL query.

The following imports : 

```js
import React from 'react'
import { graphql } from "gatsby"
```

The following gQL query (note the parameter $id passed)
```js
export const pageQuery = graphql`
  query BlogPostByPath($id: String!) {
    markdownRemark(id: {eq: $id}) {
      html
      frontmatter {
        date(formatString: "MMMM DD, YYYY")
        title
        author
      }
    }
  }
`
```

And the Component, note that the markdown parser automatically created html for us!

```js
export default function Template({ data }) {
    const post = data.markdownRemark
    
    return (
        <section>
            <span>
                <h1> {post.frontmatter.title}</h1>
                <div
                    dangerouslySetInnerHTML={{ __html: post.html }}
                />
                <h2>
                    {`${post.frontmatter.author} - ${post.frontmatter.date}`}
                </h2>
            </span>
        </section>
    )
}
```

Sweet! Now we move on to createPages in `gatsby-node.js`

#### Creating blog posts in gatsby-node.js

First, inside the exports.createPages method, we query for the paths and id's of all blog posts

```js
const result = await graphql(`
  {
    allMarkdownRemark {
      nodes {
        frontmatter {
          path
        }
        id
      }
    }
  }
`)
```

then we need to get the absolute path of the blog-post.js template

we write
```js
const template = path.resolve(`src/templates/blog-post.js`)
```

next, we create the page for each node (blog post)!
```js
result.data.allMarkdownRemark.nodes.forEach(node => {
  createPage({
    path: node.frontmatter.path,
    component: template,
    context: {
      id: node.id,
    }, // additional data can be passed via context
  })
})
```

The full gatsby-node.js is as follows: 

```js

// Create pages programmatically

const path = require("path")

exports.createPages = async ({ actions, graphql }) => {
  const { createPage } = actions
  // get the blog posts using graphQL
  const result = await graphql(`
    {
      allMarkdownRemark {
        nodes {
          frontmatter {
            path
          }
          id
        }
      }
    }
  `)
  
  // get the blog post template
  const template = path.resolve(`src/templates/blog-post.js`)

  // use the createPage() method to create pages.
  result.data.allMarkdownRemark.nodes.forEach(node => {
    createPage({
      path: node.frontmatter.path,
      component: template,
      context: {
        id: node.id,
      }, // additional data can be passed via context
    })
  })
}
```
Lets see our blog post!

Run `gatsby develop` or `npm start` and go to the correct path.
