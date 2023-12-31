<script setup lang="ts">
import Chart from '../components/Chart.vue'

// https://github.com/octokit/core.js#plugins
import { Octokit } from "@octokit/core";
import { createAppAuth } from "@octokit/auth-app"
// https://github.com/octokit/plugin-paginate-rest.js/
import { paginateRest } from "@octokit/plugin-paginate-rest";
import { ref } from 'vue'

// Octokit
const auth = createAppAuth({
  appId: import.meta.env.VITE_APP_ID,
  privateKey: import.meta.env.VITE_PRIVATE_KEY,
  clientId: import.meta.env.VITE_CLIENT_ID,
  clientSecret: import.meta.env.VITE_APP_SECRET,
});

const MyOctokit = Octokit.plugin(paginateRest);
const octokit = new MyOctokit({ auth: import.meta.env.API_TOKEN });

// Data
const openedIssues = ref([] as Items[]);
const closedIssues = ref([] as Items[]);

// Constants
const owner = 'rancher';
const repo = 'dashboard';
const labels = 'kind/bug';

const dateRanges = [
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

const getIssues = async (state: "open" | "closed", start: string, end: string) => {
  const list = await octokit.paginate(
  "GET /repos/{owner}/{repo}/issues",{
    owner,
    repo,
    labels,
      per_page: 100,
    sort: 'created',
      since: `${start}..${end}`,
    state
  },
  (response) => response.data.map(({ html_url, number, title }) => ({ html_url, number, title })),
  )
  return { date: { start, end }, list}
}

const getIssueRange = (state: "open" | "closed") => {
  return Promise.all(dateRanges.map(
    ([start, end]) => getIssues(state, formatDate(start), formatDate(end))
  ))
}

//   const stats = [
//   {date: {start: '2023-01-01', end: '2023-01-30'}, list: Array(283)},
// {date: {start: '2023-02-01', end: '2023-02-28'}, list: Array(271)},
// {date: {start: '2023-03-01', end: '2023-03-30'}, list: Array(244)},
// {date: {start: '2023-04-01', end: '2023-04-30'}, list: Array(238)},
// {date: {start: '2023-05-01', end: '2023-05-30'}, list: Array(229)},
// {date: {start: '2023-06-01', end: '2023-06-30'}, list: Array(218)},
// {date: {start: '2023-07-01', end: '2023-07-30'}, list: Array(202)},
// {date: {start: '2023-08-01', end: '2023-08-30'}, list: Array(184)},
// {date: {start: '2023-10-01', end: '2023-10-30'}, list: Array(49)},
// {date: {start: '2023-11-01', end: '2023-11-30'}, list: Array(0)},
// {date: {start: '2023-12-01', end: '2023-12-30'}, list: Array(0)},
// ]
// openedIssues.value = stats
// closedIssues.value = stats

// Init
(async () => {
  try {
    // Retrieve JSON Web Token (JWT) to authenticate as app
    const appAuthentication = await auth({ type: "app" });
    console.log(appAuthentication)

    openedIssues.value = await getIssueRange('open')
    closedIssues.value = await getIssueRange('closed')
  } catch (error) {
    // console.error(error);
  }
})()
</script>

<template>
  <main>
    <h1>Opened vs closed Bug Issues: </h1>
    <Chart :data="openedIssues.map(item => ({date: item.date.start, value: item.list.length}))"/>
    <Chart :data="closedIssues.map(item => ({date: item.date.start, value: item.list.length}))"/>

    <!-- <div
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
    </div> -->
  </main>
</template>
