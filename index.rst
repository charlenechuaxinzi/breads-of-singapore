****************************************************************************
MapLibre with Svelte
****************************************************************************

1. Introduction
=================

This is a tutorial to display a map in Svelte using MapLibre GL JS.


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
