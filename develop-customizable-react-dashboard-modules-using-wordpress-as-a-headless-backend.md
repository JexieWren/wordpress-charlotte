# Develop Customizable React Dashboard Modules Using WordPress as a Headless Backend

As I embarked on my journey as a web developer many years ago, the prospect of creating custom administrative interfaces often felt like an insurmountable challenge. Crafting modular, reusable components seemed like a skill reserved for seasoned engineers at prominent agencies.

Fast forward to today, and as a frontend developer at [Hybrid Web Agency](https://hybridwebagency.com/), I've had the privilege of working on various admin dashboards powered by headless CMS using React. The evolution of frameworks and tooling has brought us to a point where what was once considered daunting is now attainable for developers of various skill levels.

One notable observation is that while many tutorials cover the basics of setting up a React app to consume WordPress data, they often lack guidance on building reusable, future-proof solutions. They also fall short when it comes to offering best practices for code organization, state management, and scalability over time.

Having faced these challenges firsthand on client projects at Hybrid in Charlotte, I'm excited to share an extensive tutorial on constructing a fully modular admin dashboard interface. The approach I'll present focuses on separating presentational and container components, implementing custom hooks for data handling and side effects, and establishing a solid foundation for seamlessly adding new admin modules later.

Rather than simply connecting to a REST API, my aim is to equip you with knowledge on how to build an admin application that follows best practices in terms of scalability, maintainability, and extensibility. By the end of this tutorial, you'll have a deep understanding of practical techniques for structuring modular React applications powered by WordPress as a CMS. No more guesswork or reinventing the wheel with each new feature.

## Setting Up WordPress as a Headless CMS

The first step is to install WordPress. You can either download and extract the files to your local development environment or deploy WordPress on a hosting service. For the purposes of this tutorial in Charlotte, I'll be using MAMP on my local machine.

Once WordPress is installed, it's essential to enable the REST API functionality. This feature allows frontend applications to interact with WordPress data through RESTful endpoints. To enable it, navigate to Plugins > Add New and search for "REST API." Install and activate the REST API plugin.

Next, we need to set the core API permissions by adding the following code to our theme's `functions.php` file:

```php
function make_rest_request_allow_any() {
  remove filter('rest_pre_serve_request', 'rest_send_cors_headers');
}
add action('rest_api_init', 'make_rest_request_allow_any');
```

This code permits requests from any domain, which is particularly important during development when our React app resides on a different origin.

The next step is to register custom post types so that our React app can retrieve and manage specialized content types beyond the default WordPress post and page types. We'll create a custom post type called "Projects" using the following code:

```php
function create_project_post_type() {

  register post type( 'project',
    array(
      'labels' => array(
        'name' => __( 'Projects' ),
      ),
    )
  );

}
add action( 'init', 'create_project_post_type' );
```

### Building the React Admin App

To kickstart our React app, run the following command in your command line:

```bash
npx create-react-app admin-app
```

This command sets up the foundation for our React project.

Our first component, Posts.js, will be responsible for fetching and displaying post data from WordPress. We'll import the `useEffect` hook and make a GET request to the `/wp-json/wp/v2/posts` endpoint:

```jsx
import { useEffect, useState } from 'react';

function Posts() {

  const [posts, setPosts] = useState([]);

  useEffect(() => {
    fetch('/wp-json/wp/v2/posts')
      .then(res => res.json())
      .then(data => setPosts(data));
  }, []);

  return (
    <div>
      {posts.map(post => (
        <Post key={post.id} post={post} />
      )}
    </div>
  );
}
```

This provides the basics of setting up WordPress as a headless CMS and rendering its data in our React interface. In the upcoming sections, we'll incorporate functionalities for creating/updating posts and implementing routing.

### Fetching Posts Data

The `useEffect` hook enables us to fetch data from the API upon component mounting. We import `useEffect` and define state variables for the posts array and a loading state:

```js
import { useEffect, useState } from 'react';

function Posts() {

  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // API call 
  }, [])
}
```

Within the `useEffect`, we make a GET request to the WordPress REST API posts endpoint. Once the response is received, we parse the JSON data and store it in the state. This update causes the component to re-render with the newly fetched data.

```js
fetch('/wp-json/wp/v2/posts')
  .then(res => res.json())
  .then(data => {
    setPosts(data);
    setLoading(false);
  });
```

### Displaying Posts

With the posts data in hand, we can proceed to display it. We'll create a `Post` component to render each individual post. This component accepts the post as

 a prop and destructures the relevant fields.

We'll also construct the overall layout and style it using Material UI grid, cards, and typography components. This enhances the visual appeal and organization of the user interface.

We iterate over the `posts` state array within `Posts.js`, rendering a `Post` component for each post. As a result, the title, excerpt, and other details are presented in an attractive grid format.

### Adding Posts 

For the form used to add posts, we'll use the `useState` hook to keep track of input values and errors within the state. We define an initial state object and setter functions.

Validation and submission are managed by a Formik wrapper. Upon submission, it invokes a `handleSubmit` function that initiates the API request to create a new post.

We'll also implement error handling to display any errors that might occur during the API request. This approach offers a more user-friendly experience compared to generic alerts.

### Routing and Layout

We can employ React Router to define routes and navigation for our single-page app. By initializing Router and adding Route paths for the primary pages, we can effectively manage UI elements. Common components such as headers are extracted into reusable `Header.js` and `Sidebar.js` components. These elements are displayed on every route via `props.children`.

Furthermore, we can selectively present button links for adding new posts and navigating between sections by using the React Router Link component. This ties all the components together into a polished admin interface.

## Embracing Modularity

Now that we've established the fundamentals of fetching, displaying, and managing posts, it's time to enhance the modularity and reusability of our code. This involves abstracting shared logic and user interface elements into custom hooks and components.

A logical starting point is the data-fetching logic. Currently, we have `useEffect` calls directly within our `Posts` component, but this logic can be abstracted. Let's create a `useApiData` hook that handles GET requests and returns the data:

```js
function useApiData(endpoint) {

  const [data, setData] = useState([]);

  useEffect(() => {
    fetch(endpoint)
      .then(res => res.json())
      .then(setData);
  }, [endpoint]);

  return data;
}
```

Now, within `Posts`, we simply call the hook and simplify the component code:

```js 
function Posts() {

  const posts = useApiData('/posts');

  return (
    <PostList posts={posts} />
  );
}
```

This modification renders our data-fetching logic reusable across other components.

As we expand the admin's feature set, we can segment sections into logical feature modules that encapsulate related user interface and logic entirely. Let's introduce a profiles section for managing site users. We'll create a `Profiles` module with its route and reuse the data hook:

```js
function Profiles() {

  const profiles = useApiData('/profiles');

  return (
    <ProfileList profiles={profiles} />
  );
}
```

By constructing modular, reusable components, our code becomes more organized and easier to maintain as we introduce new features. Common utilities like hooks prevent code duplication throughout the application. This structure also ensures that our admin interface is highly customizable, enabling the inclusion or exclusion of modules based on specific requirements. Additional sections, such as settings, can follow the same pattern.

In summary, by prioritizing modularity and the separation of concerns, we ensure that the admin dashboard scales gracefully as requirements evolve over time, even in Charlotte at our hybrid agency.

## Wrapping Up

In this tutorial, we've delved into the architecture of a modular React admin dashboard powered by WordPress as a headless CMS. By decoupling the frontend from the backend, adhering to best practices in component structure and reusable logic, and emphasizing modularity, we've established a strong foundation for continuous improvement of the admin interface.

While the fundamentals of rendering and managing WordPress content are straightforward, the true value lies in building solutions that stand the test of time. By developing applications using techniques like custom hooks, feature modules, and the separation of concerns, websites can evolve in a maintainable way without the need for extensive refactoring.

This approach holds particular importance for organizations like us at Hybrid Web Agency, which provides customized [WordPress development services in Charlotte](https://hybridwebagency.com/charlotte-nc/custom-wordpress-development-services/). When collaborating with clients on long-term website projects, sustainability becomes a critical factor. A modular headless approach ensures that the admin dashboard and public-facing sites we build are equipped to adapt as future needs arise.

For organizations looking to optimize and extend their WordPress deployments, embracing this model provides a path to successful collaboration. Hybrid's team of WordPress experts is well-equipped to implement a production-ready version of this architecture, integrate additional features, or upgrade an existing platform. Whether you're starting from scratch or evolving your current infrastructure, harnessing headless technologies backed by robust coding practices sets the stage for success.

## References

- [WordPress REST API Handbook](https://developer.wordpress.org/rest-api/) - The official documentation for the WordPress REST API.
- [React Website for Learning React](https://reactjs.org/) - The primary resource for learning React, maintained by its creators.
- [WordPress CLI Documentation](https://developer.wordpress.org/cli/commands/) - Documentation for the WP-CLI tool used to manage WordPress.
- [GraphQL Docs](https://graphql.org/learn/) - An introductory guide to learning

 GraphQL from the maintainers.
- [Apollo Client Docs](https://www.apollographql.com/docs/react/) - Documentation for the popular Apollo GraphQL client for React.
