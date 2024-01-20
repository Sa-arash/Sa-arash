
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Profile Summary For GitHub</title>
        <link rel="icon" href="/favicon.png">
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta name="description" content="Github Profile Summary is a GitHub visualization tool written in Kotlin">
        <meta property="og:title" content="Github Profile Summary - Visualize your GitHub profile">
        <meta property="og:site_name" content="Github Profile Summary">
        <meta property="og:url" content="https://profile-summary-for-github.com">
        <meta property="og:description" content="Github Profile Summary is a GitHub visualization tool written in Kotlin">
        <meta property="og:image" content="https://user-images.githubusercontent.com/1521451/33957306-8e1d8af0-e041-11e7-8e04-3de9e32868ba.PNG">
        <meta name="google-site-verification" content="HFqNncqWzEo0ZmlYB0fnuPLSGe48YvR9BDPOrhPnXSM" />
        <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css">
        <link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Barlow+Semi+Condensed">
        <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/load-awesome@1.1.0/css/square-jelly-box.min.css">
        <script src="https://cdn.jsdelivr.net/npm/chart.js@2.8.0/dist/Chart.min.js"></script>
        <script src="https://cdn.jsdelivr.net/npm/axios@0.19.0/dist/axios.min.js"></script>
        <script src="https://cdn.jsdelivr.net/npm/vue@2.6.10/dist/vue.js"></script>
        <script src="https://cdn.jsdelivr.net/npm/moment@2.24.0/min/moment.min.js"></script>
        <script src="https://cdn.jsdelivr.net/npm/js-cookie@2/src/js.cookie.min.js"></script>
        

<script >
    class LoadableData {
        constructor(url, cache = true, errorCallback = null) {
            this._url = url;
            this._errorCallback = errorCallback;
            this.refresh(cache);
            this.addRefreshListener();
        }
        refresh(cache = true) {
            this.data = null;
            this.loading = true;
            this.loaded = false;
            this.loadError = null;
            let cacheKey = "LoadableData:" + this._url;
            if (cache) {
                this.data = JSON.parse(localStorage.getItem(cacheKey)) || null;
                this.loaded = this.data !== null;
                this.loading = this.loaded === false;
            }
            fetch(this._url).then(res => {
                if (res.ok) return res.json();
                throw JSON.stringify({code: res.status, text: res.statusText});
            }).then(data => {
                this.data = data;
                this.loaded = true;
                if (cache) {
                    localStorage.setItem(cacheKey, JSON.stringify(data));
                }
            }).catch(error => {
                this.loadError = JSON.parse(error);
                if (this._errorCallback !== null) { // should probably handle in UI
                    this._errorCallback(error);
                }
            }).finally(() => this.loading = false);
        }
        refreshAll() {
            LoadableData.refreshAll(this._url);
        }
        static refreshAll(url) {
            window.dispatchEvent(new CustomEvent("javalinvue-loadable-data-update", {detail: url}));
        }
        addRefreshListener() {
            window.addEventListener("javalinvue-loadable-data-update", e => {
                if (this._url === e.detail) {
                    this.refresh(false);
                }
            }, false);
        }
    }
</script>
<!-- search-view.vue -->
<template id="search-view">
    <app-frame v-slot="{requestsLeft}">
        <div class="search-screen">
            <h1>Enter GitHub username</h1>
            <input type="text" name="q" placeholder="ex. 'tipsy'" v-model="query" autofocus @keydown.enter="search">
            <div v-if="error && error.response.status === 404">
                <h4>Can't find user <span class="search-term">{{failedQuery}}</span>. Check spelling.</h4>
            </div>
            <div v-else-if="failedQuery">
                <h4>Can't build profile for <span class="search-term">{{failedQuery}}</span></h4>
                <p>
                    If you are <span class="search-term">{{failedQuery}}</span>, please
                    <a href="https://github.com/tipsy/profile-summary-for-github">star the repo</a> and try again.
                </p>
                <p>
                    The app is running with two GitHub tokens, giving 10 000 requests per hour.
                    The first 5000 requests can be used to build any profile, while the last 5000 requests are
                    reserved for users building their own profile. To confirm that you're building your own
                    profile, we check if you've starred the repository.
                </p>
            </div>
            <div v-if="requestsLeft === 0">
                The app is rate limited. Please come back later or build the app locally and use your own tokens.
            </div>
        </div>
    </app-frame>
</template>
<script>
    Vue.component("search-view", {
        template: "#search-view",
        data: () => ({
            error: null,
            failedQuery: "",
            query: ""
        }),
        methods: {
            search() {
                this.error = null;
                this.failedQuery = null;
                axios.get("/api/can-load?user=" + this.query)
                    .then(() => window.location = "/user/" + this.query)
                    .catch(error => {
                        this.error = error;
                        this.failedQuery = this.query
                    });
            }
        },
    });
</script>
<style>
    .search-screen {
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .search-term {
        border: 1px solid rgba(0, 0, 0, 0.2);
        background: rgba(0, 0, 0, 0.025);
        padding: 1px 2px;
        font-family: monospace;
        font-size: 80%;
    }

    .search-screen input {
        height: 40px;
        font-size: 18px;
        padding: 0 15px;
        border: 0;
    }
</style>

<!-- share-bar.vue -->
<template id="share-bar">
    <div class="share-bar">
        <a class="social-btn" :href="twitterUrl" rel="nofollow" title="Share on Twitter"><i class="fa fa-fw fa-twitter"></i>Share on Twitter</a>
        <a class="social-btn" :href="facebookUrl" rel="nofollow" title="Share on Facebook"><i class="fa fa-fw fa-facebook"></i>Share on Facebook</a>
    </div>
</template>
<script>
    Vue.component("share-bar", {
        template: "#share-bar",
        props: ["user"],
        computed: {
            profileUrl: function () {
                return "https://profile-summary-for-github.com/user/" + this.user.login;
            },
            shareText: function () {
                return this.user.login + "'s GitHub profile - Visualized:";
            },
            twitterUrl: function () {
                return "https://twitter.com/intent/tweet?url=" + this.profileUrl + "&text=" + this.shareText + "&via=javalin_io&related=javalin_io";
            },
            facebookUrl: function () {
                return "https://facebook.com/sharer.php?u=" + this.profileUrl + "&quote=" + this.shareText
            }
        }
    });
</script>
<style>
    .share-bar {
        position: absolute;
        top: 0;
        left: 50%;
        transform: translateX(-50%);
        background: rgba(0, 0, 0, .04);
        font-size: 14px;
        text-align: center;
    }

    .share-bar a {
        white-space: nowrap;
        margin: 5px 8px;
        display: inline-block;
    }

    .share-bar a i {
        color: #0082c8;
    }

    @media (max-width: 480px) {
        .share-bar {
            width: 100%;
        }
    }
</style>

<!-- user-info.vue -->
<template id="user-info">
    <div class="user-info">
        <img :src="user.avatarUrl" :alt="user.login">
        <div class="details">
            <div><i class="fa fa-fw fa-user"></i>{{ user.login }}
                <small v-if="user.name">({{ user.name }})</small>
            </div>
            <div><i class="fa fa-fw fa-database"></i>{{ user.publicRepos }} public repos</div>
            <div><i class="fa fa-fw fa-clock-o"></i>Joined GitHub {{ timeAgo }}</div>
            <div v-if="user.email"><i class="fa fa-fw fa-envelope"></i> {{ user.email }}</div>
            <div v-if="user.company"><i class="fa fa-fw fa-building"></i>{{ user.company }}</div>
            <div><i class="fa fa-fw fa-external-link"></i><a :href="user.htmlUrl" target="_blank">View profile on GitHub</a></div>
        </div>
        <div class="chart-container commits-per-quarter">
            <canvas id="quarterCommitCount"></canvas>
        </div>
    </div>
</template>
<script>
    Vue.component("user-info", {
        template: "#user-info",
        props: ["user", "data"],
        computed: {
            timeAgo() {
                return moment(this.user.createdAt).fromNow()
            }
        },
        mounted() {
            lineChart("quarterCommitCount", this.data)
        }
    });
</script>
<style>
    .user-info {
        display: flex;
        padding-bottom: 40px;
    }

    .user-info img {
        align-self: center;
        border-radius: 3px;
        width: 175px;
        margin-right: 20px;
    }

    .user-info .details {
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        margin-right: 20px;
        flex-shrink: 0;
    }

    .user-info i.fa {
        color: rgba(0, 0, 0, 0.67);
        margin-right: 5px;
    }

    .user-info .commits-per-quarter {
        flex-grow: 1;
        flex-shrink: 1;
        position: relative;
    }

    .user-info .commits-per-quarter::after {
        content: "Commits per quarter";
        position: absolute;
        right: 40px;
        bottom: -15px;
        font-size: 13px;
    }

    @media (max-width: 480px) {
        .user-info img,
        .user-info .commits-per-quarter{
            display: none;
        }
    }
</style>

<!-- donut-charts.vue -->
<template id="donut-charts">
    <div class="charts">
        <div class="chart-row">
            <div class="chart-container chart-container--third">
                <h2>Repos per Language</h2>
                <canvas id="langRepoCount"></canvas>
            </div>
            <div v-if="Math.max(...Object.values(data.repoStarCount)) > 0" class="chart-container chart-container--third">
                <h2>Stars per Language</h2>
                <canvas id="langStarCount"></canvas>
            </div>
            <div class="chart-container chart-container--third">
                <h2>Commits per Language</h2>
                <canvas id="langCommitCount"></canvas>
            </div>
        </div>
        <div class="chart-row">
            <div class="chart-container chart-container--half">
                <h2>Commits per Repo
                    <small v-if="Object.keys(data.repoCommitCount).length === 10">(top 10)</small>
                </h2>
                <canvas id="repoCommitCount"></canvas>
            </div>
            <div v-if="Object.keys(data.repoStarCount).length > 0" class="chart-container chart-container--half">
                <h2>Stars per Repo
                    <small v-if="Object.keys(data.repoStarCount).length == 10">(top 10)</small>
                </h2>
                <canvas id="repoStarCount"></canvas>
            </div>
        </div>
    </div>
</template>
<script>
    Vue.component("donut-charts", {
        template: "#donut-charts",
        props: ["data"],
        mounted() {
            donutChart("langRepoCount", this.data);
            donutChart("langStarCount", this.data);
            donutChart("langCommitCount", this.data);
            donutChart("repoCommitCount", this.data);
            donutChart("repoStarCount", this.data);
        }
    });
</script>
<style>
    canvas {
        user-select: none;
    }

    .charts,
    .chart-row {
        overflow: auto;
    }

    .chart-row {
        padding-bottom: 40px;
    }

    .chart-row {
        border-top: 1px solid rgba(0, 0, 0, 0.1);
        display: flex;
        justify-content: space-around;
    }

    .chart-container--third {
        width: 33%;
    }

    .chart-container--half {
        width: 50%;
    }

    @media (max-width: 900px) {
        .chart-container--third,
        .chart-container--half {
            width: 100%;
        }

        .chart-row {
            display: block;
        }

    }

    @media (max-width: 480px) {
        footer {
            display: none;
        }
    }
</style>

<!-- _main-styles.vue -->
<style>
    * {
        font-family: 'Barlow Semi Condensed', sans-serif;
        outline: 0;
        box-sizing: border-box;
    }

    [v-cloak] {
        display: none;
    }

    html {
        font-size: 18px;
        background: #eee9df;
        padding: 60px 30px;
        overflow-y: scroll;
    }

    body {
        margin: 0;
    }

    h1, h2, h3, h4 {
        font-weight: 400;
    }

    a {
        color: #0082c8;
        text-decoration: none;
    }

    .content {
        max-width: 1200px;
        margin: 0 auto;
    }

    .fade-in {
        opacity: 0;
        animation: fade-in .2s linear forwards;
    }

    @keyframes fade-in {
        from {
            opacity: 0;
        }
        to {
            opacity: 1;
        }
    }
</style>

<!-- loading-bouncer.vue -->
<template id="loading-bouncer">
    <div class="loading-bouncer" style="opacity: 0; animation: fade-in 0.2s linear 0.5s forwards">
        <div class="la-square-jelly-box la-3x">
            <div></div>
            <div></div>
        </div>
        <h2>Analyzing GitHub profile</h2>
        <h3 style="opacity: 0; animation: fade-in 0.2s linear 4s forwards">This could take some time ...</h3>
        <h3 style="opacity: 0; animation: fade-in 0.2s linear 8s forwards">This user has a lot of repos!</h3>
    </div>
</template>
<script>
    Vue.component("loading-bouncer", {template: "#loading-bouncer"});
</script>
<style>
    .loading-bouncer {
        margin-top: 50px;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .la-square-jelly-box {
        color: #38abe2;
    }
</style>

<!-- _charts.vue -->
<script>
    const UNKNOWN_LANGUAGE = "Unknown";

    function donutChart(objectName, data) {
        let canvas = document.getElementById(objectName);
        if (canvas === null) {
            return;
        }
        let userId = data.user.login;
        let labels = Object.keys(data[objectName]);

        /**
         * The language index of <code>UNKNOWN_LANGUAGE</code>
         * @type {Number}
         */
        let indexOfUnknownLanguage = -1;
        let values = Object.values(data[objectName]);
        let colors = createColorArray(labels.length);
        let tooltipInfo = null;
        window.languageColors = window.languageColors || {};
        if ("langRepoCount" === objectName) {
            // when the first language-set is loaded, set a color-profile for all languages
            labels.forEach((language, i) => languageColors[language] = colors[i]);
        }
        if (["langRepoCount", "langStarCount", "langCommitCount"].indexOf(objectName) > -1) {
            // if the dataset is language-related, load color-profile
            labels.forEach((language, i) => colors[i] = languageColors[language]);
            indexOfUnknownLanguage = labels.indexOf(UNKNOWN_LANGUAGE);
        }
        if (objectName === "repoCommitCount") {
            tooltipInfo = data[objectName + "Descriptions"]; // high quality programming
            arrayRotate(colors, 4); // change starting color
        }
        if (objectName === "repoStarCount") {
            tooltipInfo = data[objectName + "Descriptions"]; // high quality programming
            arrayRotate(colors, 2); // change starting color
        }
        new Chart(canvas.getContext("2d"), {
            type: "doughnut",
            data: {
                labels: labels,
                datasets: [{
                    data: values,
                    backgroundColor: colors
                }]
            },
            options: {
                animation: false,
                rotation: (-0.40 * Math.PI),
                legend: { // todo: fix duplication ?
                    position: window.innerWidth < 600 ? "bottom" : "left",
                    labels: {
                        fontSize: window.innerWidth < 600 ? 10 : 12,
                        padding: window.innerWidth < 600 ? 8 : 10,
                        boxWidth: window.innerWidth < 600 ? 10 : 12
                    }
                },
                tooltips: {
                    callbacks: {
                        afterLabel: function (tooltipItem, data) {
                            if (tooltipInfo !== null) {
                                return wordWrap(tooltipInfo[data["labels"][tooltipItem["index"]]], 45);
                            }

                            if (tooltipItem.index === indexOfUnknownLanguage) {
                                return "No specific language";
                            }
                        }
                    },
                },
                onClick: function (e, data) {
                    try {
                        let label = labels[data[0]._index];
                        const isUnknownLanguage = (data[0]._index === indexOfUnknownLanguage);
                        let canvas = data[0]._chart.canvas.id;
                        if (canvas === "repoStarCount" || canvas === "repoCommitCount") {
                            window.open("https://github.com/" + userId + "/" + label, "_blank");
                            window.focus();
                        } else {
                            if (!isUnknownLanguage) {
                                window.open("https://github.com/" + userId + "?utf8=%E2%9C%93&tab=repositories&q=&type=source&language=" + encodeURIComponent(label), "_blank");
                                window.focus();
                            }
                        }
                    } catch (ignored) {
                    }
                },
                onResize: function (instance) { // todo: fix duplication ?
                    instance.chart.options.legend.position = window.innerWidth < 600 ? "bottom" : "left";
                    instance.chart.options.legend.labels.fontSize = window.innerWidth < 600 ? 10 : 12;
                    instance.chart.options.legend.labels.padding = window.innerWidth < 600 ? 8 : 10;
                    instance.chart.options.legend.labels.boxWidth = window.innerWidth < 600 ? 10 : 12;
                }
            }
        });

        function createColorArray(length) {
            const colors = ["#54ca76", "#f5c452", "#f2637f", "#9261f3", "#31a4e6", "#55cbcb"];
            let array = [...Array(length).keys()].map(i => colors[i % colors.length]);
            // avoid first and last colors being the same
            if (length % colors.length === 1) {
                array[length - 1] = colors[1];
            }
            return array;
        }

        function arrayRotate(arr, n) {
            for (let i = 0; i < n; i++) {
                arr.push(arr.shift());
            }
            return arr
        }

        function wordWrap(str, n) {
            if (str === null) {
                return null;
            }
            let currentLine = [];
            let resultLines = [];
            str.split(" ").forEach(word => {
                currentLine.push(word);
                if (currentLine.join(" ").length > n) {
                    resultLines.push(currentLine.join(" "));
                    currentLine = [];
                }
            });
            if (currentLine.length > 0) {
                resultLines.push(currentLine.join(" "));
            }
            return resultLines
        }
    }

    function lineChart(objectName, data) {
        new Chart(document.getElementById(objectName).getContext("2d"), {
            type: "line",
            data: {
                labels: Object.keys(data[objectName]),
                datasets: [{
                    label: "Commits",
                    data: Object.values(data[objectName]),
                    backgroundColor: "rgba(67, 142, 233, 0.2)",
                    borderColor: "rgba(67, 142, 233, 1)",
                    lineTension: 0
                }]
            },
            options: {
                maintainAspectRatio: false,
                animation: false,
                scales: {
                    xAxes: [{
                        display: false
                    }],
                    yAxes: [{
                        position: "right",
                        beginAtZero: true
                    }]
                },
                legend: {
                    display: false
                },
                tooltips: {
                    intersect: false
                }
            }
        });
    }
</script>

<!-- app-frame.vue -->
<template id="app-frame">
    <div>
        <main class="main-content">
            <slot :requests-left="requestsLeft"></slot>
        </main>
        <footer>
            GitHub profile summary is built with <a href="https://javalin.io">Javalin 5.0.0</a> <small>(kotlin web framework)</small> and
            <a href="http://www.chartjs.org/docs/latest/" target="_blank">chart.js</a> <small>(visualization)</small>.
            Source is on <a href="https://github.com/tipsy/profile-summary-for-github" target="_blank">GitHub</a>.
        </footer>
        <span class="rate-limit">
            <span v-if="requestsLeft === 0">The app is currently rate-limited<br>Please check back later</span>
            <span v-if="requestsLeft !== 0"><strong>{{requestsLeft}}</strong> requests left <br> before rate-limit</span>
        </span>
    </div>
</template>
<script>
    Vue.component("app-frame", {
        template: "#app-frame",
        data: () => ({
            requestsLeft: 9876,
        }),
        created() {
            let wsProtocol = window.location.protocol.indexOf("https") > -1 ? "wss" : "ws";
            let ws = new WebSocket(wsProtocol + "://" + location.hostname + ":" + location.port + "/rate-limit-status");
            ws.onmessage = msg => this.requestsLeft = msg.data;
        },
    });
</script>
<style>
    footer {
        font-size: 17px;
        position: fixed;
        left: 0;
        bottom: 0;
        width: 100%;
        text-align: center;
        padding: 10px 30px;
        border-top: 1px solid rgba(0, 0, 0, 0.1);
        background: #eee9df;
    }

    .rate-limit {
        position: fixed;
        right: 20px;
        bottom: 60px;
        background: #fff;
        padding: 10px;
        box-shadow: 0 1px 1px rgba(0, 0, 0, 0.3);
        font-size: 16px;
    }

    @media (max-width: 480px) {
        .rate-limit {
            padding: 5px 8px;
            font-size: 13px;
            bottom: 0;
            right: 0;
        }
    }
</style>

<!-- _gtm.vue -->
<script>
    (function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
    new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
    j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
    '//www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
    })(window,document,'script','dataLayer',Cookies.get("gtm-id"));
</script>

<!-- user-view.vue -->
<template id="user-view">
    <app-frame v-slot="{requestsLeft}">
        <loading-bouncer v-if="!data && !error"></loading-bouncer>
        <div v-if="data" class="fade-in">
            <share-bar :user="user"></share-bar>
            <user-info :user="user" :data="data"></user-info>
            <donut-charts :data="data"></donut-charts>
        </div>
        <div v-if="error">
            <div v-if="error.response.status === 404">User not found.</div>
            <div v-else-if="requestsLeft >= 5000">Something went wrong. Please try again.</div>
            <div v-else-if="requestsLeft < 5000">Less than 5000 requests left. Please star the repo and try again.</div>
            <div v-else-if="requestsLeft === 0">The app is rate-limited. Please come back later.</div>
        </div>
    </app-frame>
</template>
<script>
    Vue.component("user-view", {
        template: "#user-view",
        data: () => ({
            data: null,
            user: null,
            error: null,
        }),
        created() {
            let userId = this.$javalin.pathParams["user"];
            axios.get("/api/user/" + userId).then(response => {
                this.data = response.data;
                this.user = response.data.user;
            }).catch(error => this.error = error);
        },
    });
</script>
<script >
    Vue.prototype.$javalin = JSON.parse(decodeURIComponent('%7B%22pathParams%22%3A%7B%22user%22%3A%22Sa-arash%22%7D%2C%22state%22%3A%7B%7D%7D'))
</script>
    </head>
    <body>
        <main id="main-vue" class="content" v-cloak>
            <user-view></user-view>
        </main>
        <script>
            new Vue({el: "#main-vue"});
        </script>
    </body>
</html>
