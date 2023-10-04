<script setup lang="ts">
import { Octokit } from "@octokit/core";
import { ref } from 'vue'

const openedIssues = ref({});

const octokit = new Octokit({ auth: import.meta.env.API_TOKEN });
const owner = 'rancher';
const repo = 'dashboard';

(async () => {
  const resp = await octokit.request(`GET /repos/{owner}/{repo}/issues`, {
    owner,
    repo,
    per_page: 100,
    since: '2023-10-01',
    sort: 'created',
    labels: 'kind/bug',
  });

  openedIssues.value = resp.data.map(({url, number, title}) => ({
    url, number, title
  }));
})()
</script>

<template>
  <main>
    <h1>Opened Bug Issues: {{ openedIssues.length }}</h1>
    <ul>
      <li v-for="issue of openedIssues">
        <a :href="issue.url" target="_blank">
          #{{ issue.number }}: {{ issue.title }}
        </a>
      </li>
    </ul>
  </main>
</template>
