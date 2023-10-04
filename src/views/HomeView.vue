<script setup lang="ts">
// https://github.com/octokit/core.js#plugins
import { Octokit } from "@octokit/core";
// https://github.com/octokit/plugin-paginate-rest.js/
import { paginateRest } from "@octokit/plugin-paginate-rest";
import { ref } from 'vue'

// Octokit
const MyOctokit = Octokit.plugin(paginateRest);
const octokit = new MyOctokit({ auth: import.meta.env.API_TOKEN });

// Types
interface Issues { html_url: string; number: number; title: string; }

interface Items {
  date: { start: string; end: string; };
  list: Issues[];
}[]

// Data
const openedIssues = ref([] as Items[]);

// Constants
const owner = 'rancher';
const repo = 'dashboard';
const labels = 'kind/bug';

const dates = [
  ['2023-01-01', '2023-01-30'],
  ['2023-02-01', '2023-02-28'],
  ['2023-03-01', '2023-03-30'],
  ['2023-04-01', '2023-04-30'],
  ['2023-05-01', '2023-05-30'],
  ['2023-06-01', '2023-06-30'],
  ['2023-07-01', '2023-07-30'],
  ['2023-08-01', '2023-08-30'],
  ['2023-09-01', '2023-09-30'],
  ['2023-10-01', '2023-10-30'],
  ['2023-11-01', '2023-11-30'],
  ['2023-12-01', '2023-12-30'],
]

// Functions
const formatDate = (date?: string) => (date ? new Date(date) : new Date()).toJSON().slice(0, 10);

const getOpenedIssues = async (start: string, end: string) => {
  const list = await octokit.paginate(
  "GET /repos/{owner}/{repo}/issues",{
    owner,
    repo,
    labels,
      per_page: 100,
    sort: 'created',
    since: `${start}..${end}`,
  },
  (response) => response.data.map(({ html_url, number, title }) => ({ html_url, number, title })),
  )
  openedIssues.value.push({ date: { start, end }, list})
}

// Init
(async () => {
  openedIssues.value = []

  return dates.forEach(([start, end]) => getOpenedIssues(formatDate(start), formatDate(end)))
})()
</script>

<template>
  <main>
    <h1>Opened Bug Issues: </h1>
    <div
      v-for="item of openedIssues"
    >
      <details>
      <summary>({{ item.list.length }}) {{ item.date.start }} - {{ item.date.end }}</summary>
      <ul>
        <li
          v-for="issue of item.list"
        >
          <a :href="issue.html_url" target="_blank">
            #{{ issue.number }}: {{ issue.title }}
          </a>
        </li>
      </ul>
    </details>
    </div>
  </main>
</template>
