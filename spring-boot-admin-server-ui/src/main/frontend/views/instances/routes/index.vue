<!--
  - Copyright 2014-2018 the original author or authors.
  -
  - Licensed under the Apache License, Version 2.0 (the "License");
  - you may not use this file except in compliance with the License.
  - You may obtain a copy of the License at
  -
  -     http://www.apache.org/licenses/LICENSE-2.0
  -
  - Unless required by applicable law or agreed to in writing, software
  - distributed under the License is distributed on an "AS IS" BASIS,
  - WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  - See the License for the specific language governing permissions and
  - limitations under the License.
  -->

<template>
  <section class="section" :class="{ 'is-loading' : !hasLoaded }">
    <div class="container">
      <div v-if="error" class="message is-danger">
        <div class="message-body">
          <strong>
            <font-awesome-icon class="has-text-danger" icon="exclamation-triangle"/>
            Fetching routes failed.
          </strong>
          <p v-text="error.message"/>
        </div>
      </div>
      <div class="field has-addons" v-if="routes">
        <p class="control is-expanded">
          <input class="input" type="search" placeholder="name / value filter" v-model="filter">
        </p>
      </div>
      <sba-panel :header-sticks-below="['#navigation', '#instance-tabs']"
                 v-for="route in filteredRoutes"
                 :key="route.obj.id"
                 :title=" route.obj.id">
        <table class="table is-fullwidth">
          <tr v-if="route.obj.path">
            <td>path</td>
            <td v-text="route.obj.path"/>
          </tr>
          <tr v-if="route.obj.fullPath">
            <td>fullPath</td>
            <td v-text="route.obj.fullPath"/>
          </tr>
          <tr v-if="route.obj.prefix">
            <td>prefix</td>
            <td v-text="route.obj.prefix"/>
          </tr>
          <tr v-if="route.obj.prefix">
            <td>prefixStripped</td>
            <td v-text="route.obj.prefixStripped"/>
          </tr>
          <tr>
            <td>retryable</td>
            <td v-text="route.obj.retryable"/>
          </tr>
          <tr>
            <td>location</td>
            <td v-text="route.obj.location"/>
          </tr>
          <tr v-if="route.obj.customSensitiveHeaders">
            <td>sensitiveHeaders</td>
            <td v-text="route.obj.sensitiveHeaders"/>
          </tr>
        </table>
      </sba-panel>
    </div>
  </section>
</template>

<script>
  import Instance from '@/services/instance';

  const buildRoutesObjs = (routes) => {
    const routesObjs = [];
    const routeNames = Object.keys(routes);

    for (const routeName of routeNames) {
      const route = routes[routeName];
        routesObjs.push({
          name: routeName,
          obj: route
        });
    }

    return routesObjs;
  };

  export default {
    props: {
      instance: {
        type: Instance,
        required: true
      }
    },
    data: () => ({
      hasLoaded: false,
      error: null,
      routes: null,
      filter: null
    }),
    computed: {
      filteredRoutes() {
        if (!this.routes) {
          return [];
        }
        const routesArray = buildRoutesObjs(this.routes);
        if (!this.filter) {
          return routesArray;
        }
      return routesArray
          .filter(route => route && route.name.indexOf(this.filter) > 0);
      }
    },
    created() {
      this.fetchRoutes();
    },
    methods: {
      async fetchRoutes() {
        this.error = null;
        try {
          const routes = await this.instance.fetchRoutes();
          this.routes = routes.data;
        } catch (error) {
          console.warn('Fetching configuration properties failed:', error);
          this.error = error;
        }
        this.hasLoaded = true;
      }
    },
    install({viewRegistry}) {
      viewRegistry.addView({
        name: 'instances/routes',
        parent: 'instances',
        path: 'routes',
        component: this,
        label: 'Gateway Routes',
        order: 1000,
        isEnabled: ({instance}) => instance.hasEndpoint('routes')
      });
    }
  }
</script>
