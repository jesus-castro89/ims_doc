---
weight: 403
title: "Vistas"
description: ""
icon: "article"
date: "2024-06-24T21:05:47-06:00"
lastmod: "2024-06-24T21:05:47-06:00"
draft: false
toc: true
---

## Creando la configuración de nuestras vistas

Actualiza el archivo `package.json` con el siguiente contenido:

```json
{
  "name": "cms_ims",
  "packageManager": "yarn@4.3.1",
  "devDependencies": {
	"@babel/core": "^7.22.8",
	"@babel/plugin-transform-class-properties": "^7.24.7",
	"@babel/preset-env": "^7.22.7",
	"@symfony/webpack-encore": "^4.3.0",
	"babel-loader": "^9.1.3",
	"mini-css-extract-plugin": "^2.7.6",
	"pnp-webpack-plugin": "^1.7.0",
	"webpack": "^5.88.1",
	"webpack-cli": "^5.1.4",
	"webpack-manifest-plugin": "^5.0.0",
	"webpack-notifier": "1.15.0"
  },
  "scripts": {
	"dev-server": "encore dev-server",
	"dev": "encore dev",
	"watch": "encore dev --watch",
	"build": "encore production --progress"
  },
  "dependencies": {
	"@fortawesome/fontawesome-free": "^6.5.2",
	"@popperjs/core": "^2.11.8",
	"bootstrap": "5.3.3",
	"core-js": "^3.37.1",
	"gridjs": "^6.2.0",
	"gridjs-jquery": "^4.0.0",
	"jquery": "^3.7.1"
  }
}
```

Actualiza el archivo `assets/app.js` con el siguiente contenido:

```js
//JS
import 'bootstrap/dist/js/bootstrap';
//CSS
import 'bootstrap/dist/css/bootstrap.css';
import '@fortawesome/fontawesome-free/css/all.css';
import './styles/app.css';
```

Crea el archivo `assets/category.js` y pega el siguiente contenido:

```js
//JS
import "gridjs-jquery";
import "./scripts/categoryTable.js";
//CSS
import "gridjs/dist/theme/mermaid.css";
import "./styles/category.css";
```

Crea el archivo `assets/scripts/categoryTable.js` y pega el siguiente contenido:

```js
import {table_search, table_pagination, table_label, configTable} from './tables';
import {html} from "gridjs";

$(function () {

	configTable(['Nombre', 'Padre', 'Acciones'], data => data.results.map(entity => [
		entity.name, entity.father, html(entity.edit)
	]));
});
```

Crea el archivo `assets/scripts/tables.js` y pega el siguiente contenido:

```js
let table = $("#wrapper");

let loader = $("#table-loader").attr('loader');

let form = $('#entityForm');

let entity = $('#idEntity');

let table_label = {
	search: {
		placeholder: '🔍 Buscar...',
	},
	sort: {
		sortAsc: 'Orden de columna ascendente.',
		sortDesc: 'Orden de columna descendente.',
	},
	pagination: {
		previous: '⬅️',
		next: '➡️',
		navigate: (page, pages) => `Página ${page} de ${pages}`,
		page: (page) => `Página ${page}`,
		showing: 'Mostrando del',
		of: 'de',
		to: 'al',
		results: 'registros',
	},
	loading: 'Cargando...',
	noRecordsFound: 'No se encontraron resultados.',
	error: 'Ocurrió un error al cargar los datos.',
};

let table_pagination = {
	limit: 10,
	server: {
		url: (prev, page, limit) => `${prev}/${limit}/${page * limit}`
	}
}

let table_search = {
	server: {
		url: (prev, keyword) => `${prev}/${keyword}`
	}
};

function configTable(columns, server_data) {
	table.Grid({
		columns: columns,
		pagination: table_pagination,
		search: table_search,
		server: {
			url: loader,
			then: server_data,
			total: data => data.count
		},
		language: table_label
	});

	table.on('click', '.edit', function () {

		entity.val($(this).attr('entity'));
		form.attr('action', $('#editLink').val());
		form.trigger("submit")
	});
}

export {configTable};
```

## Actualizar el archivo de Webpack

Abre el archivo `webpack.config.js` y agrega la siguiente configuración:

```js
const Encore = require('@symfony/webpack-encore');

// Manually configure the runtime environment if not already configured yet by the "encore" command.
// It's useful when you use tools that rely on webpack.config.js file.
if (!Encore.isRuntimeEnvironmentConfigured()) {
	Encore.configureRuntimeEnvironment(process.env.NODE_ENV || 'dev');
}

Encore
	// directory where compiled assets will be stored
	.setOutputPath('public/build/')
	// public path used by the web server to access the output path
	.setPublicPath('http://localhost/cms_ims/build')
	.configureFilenames({
		js: '[name].js',
		css: '[name].css',
		assets: 'assets/[name].[ext]',
	})
	// only needed for CDN's or subdirectory deploy
	.setManifestKeyPrefix('build/')

	/*
     * ENTRY CONFIG
     *
     * Each entry will result in one JavaScript file (e.g. app.js)
     * and one CSS file (e.g. app.css) if your JavaScript imports CSS.
     */
	.addEntry('app', './assets/app.js')
	.addEntry('category', './assets/category.js')

	// When enabled, Webpack "splits" your files into smaller pieces for greater optimization.
	.splitEntryChunks()

	// will require an extra script tag for runtime.js
	// but, you probably want this, unless you're building a single-page app
	.enableSingleRuntimeChunk()
	.addLoader({
		test: /\.js$/,
		loader: require.resolve('babel-loader'),
	})


	/*
     * FEATURE CONFIG
     *
     * Enable & configure other features below. For a full
     * list of features, see:
     * https://symfony.com/doc/current/frontend.html#adding-more-features
     */
	.cleanupOutputBeforeBuild()
	.enableBuildNotifications()
	.enableSourceMaps(!Encore.isProduction())
	// enables hashed filenames (e.g. app.abc123.css)
	.enableVersioning(Encore.isProduction())

	.configureBabel((config) => {
		config.plugins.push('@babel/plugin-transform-class-properties');
	})

	// enables @babel/preset-env polyfills
	.configureBabelPresetEnv((config) => {
		config.useBuiltIns = 'usage';
		config.corejs = 3;
	})

	// enables Sass/SCSS support
	//.enableSassLoader()

	// uncomment if you use TypeScript
	//.enableTypeScriptLoader()

	// uncomment if you use React
	//.enableReactPreset()

	// uncomment to get integrity="..." attributes on your script & link tags
	// requires WebpackEncoreBundle 1.4 or higher
	//.enableIntegrityHashes(Encore.isProduction())

	// uncomment if you're having problems with a jQuery plugin
	.autoProvidejQuery()
;

module.exports = Encore.getWebpackConfig();
```

Ejecuta el siguiente comando para compilar los archivos:

```bash
yarn dev
```