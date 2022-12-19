****************************************************************************
MapLibre with Svelte
****************************************************************************

1. Introduction
=================

This is a tutorial to display a map in Svelte using MapLibre GL JS.
Based on the amptiler tutorial: https://docs.maptiler.com/svelte/maplibre-gl-js/how-to-use-maplibre-gl-js/


2. Create an app
=================

First, we are going to create a Svelte app.
To create a new Svelte project, run in the command-line:

.. code-block:: console

  npx degit sveltejs/template my-svelte-map



Now navigate to the newly created project folder **my-svelte-map**:


.. code-block:: console

  cd my-svelte-map


Inside this folder, run ``npm install`` to install the dependencies.

To start the local environment, run ``npm run dev``. An type the address on your prefered browser: ``http://localhost:5000``.

You'll see the app in your browser.


3. Install MapLibre and set up
================================

To install MapLibre GL, navigate to the project folder and run the command:

.. code-block:: console

  npm i maplibre-gl


Navigate to the ``src`` folder and delete the props name of the ``main.js`` file. Your ``main.js`` file should look like this:

.. code-block:: javascript

  import App from './App.svelte';

  const app = new App({
  target: document.body,
  props: {}
  });

  export default app;


Delete all the content of the App.svelte file and write the following lines in the App.svelte file

.. code-block:: javascript

  <script>

  </script>

  <div class="app">
    This is my map App
  </div>

  <style>
  .app {
    text-align: center;
  }
  </style>

You shold visualize the changes in your browser.


4. Create a navbar component
================================

In this step, we will create a simple heading navbar component.

Create a new folder called ``components`` indide the ``src`` folder.

Create a new file called ``Navbar.svelte`` inside the ``components`` folder and write these lines:

.. code-block:: javascript

  <div class="heading">
  <h1>This is my map App</h1>
  </div>

  <style>
  .heading {
    margin: 0;
    padding: 0px;
    background-color: black;
    color: white;
  }

  .heading > h1 {
    padding: 20px;
    margin: 0;
  }
  </style>


Finally, to display the **Navbar** we need to import the Navbar component and add it to our main component ``App.svelte``.

Import the navbar component into ``App.svelte`` script block:


.. code-block:: javascript

  <script>
    import Navbar from './components/Navbar.svelte';
  </script>

Replace the text *This is my map App** with ``<Navbar/>``. Your ``App.svelte`` file should look like this:

.. code-block:: javascript

  <script>
    import Navbar from './components/Navbar.svelte';
  </script>

  <div class="app">
    <Navbar />
  </div>

  <style>
    .app {
      text-align: center;
    }
  </style>
