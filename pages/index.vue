<template>
    <div class="container px-4 mx-auto min-h-screen">

        <div v-if="isOpen">
            <ServerModal :isOpenModal="isOpen"></ServerModal>
        </div>

        <div class="text-white my-4 min-h-[20vh] flex items-center">
            <h1 class="text-4xl font-extrabold">
                Your Favourite <br>
                <span class="text-purple-500 flex items-center">Anime
                    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-8 h-8 ml-2">
                        <path fill-rule="evenodd"
                            d="M12.963 2.286a.75.75 0 00-1.071-.136 9.742 9.742 0 00-3.539 6.177A7.547 7.547 0 016.648 6.61a.75.75 0 00-1.152-.082A9 9 0 1015.68 4.534a7.46 7.46 0 01-2.717-2.248zM15.75 14.25a3.75 3.75 0 11-7.313-1.172c.628.465 1.35.81 2.133 1a5.99 5.99 0 011.925-3.545 3.75 3.75 0 013.255 3.717z"
                            clip-rule="evenodd" />
                    </svg>
                </span>
            </h1>
        </div>
        <div class="flex overflow-x-auto w-full gap-3 py-4">
            <div v-for="g, k in Genres" :key="k" class="">
                <div class="text-xs px-4 py-2 rounded-full bg-purple-500 border-purple-500 text-white whitespace-nowrap cursor-pointer"
                    v-if="k == 'all'">
                    {{ g }}
                </div>
                <NuxtLink v-if="k != 'all'" :to="'/genre/' + k + '?page=1'"
                    class="text-xs px-4 py-2 rounded-full bg-purple-900 border-purple-500 text-white whitespace-nowrap cursor-pointer">
                    {{ g }}
                </NuxtLink>
            </div>
        </div>
        <div class="my-5">
            <div class="flex justify-between text-white">
                <h1 class="mb-4 text-xl">Recent Release</h1>
                <NuxtLink class="text-xs flex items-center" to="/recent-release?page=1">View More <svg
                        xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
                        stroke="currentColor" class="w-4 h-4 ml-2">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M8.25 4.5l7.5 7.5-7.5 7.5" />
                    </svg>
                </NuxtLink>
            </div>
            <div v-if="recentRelease != null && recentRelease.length > 0"
                class="flex overflow-x-auto gap-5 w-full snap-x scroll-smooth">
                <RecentReleaseComponent v-for="rr in recentRelease" :key="rr" :release="rr" class="snap-start" />
            </div>
            <div v-else class="flex overflow-x-auto gap-5 w-full snap-x scroll-smooth">
                <!-- <SpiningLoading></SpiningLoading> -->
                <LoadingAnimeCard v-for="i in 5" :key="i"></LoadingAnimeCard>
            </div>
        </div>

        <div class="mt-5 pb-8" v-if="server == 'gogoanime'">
            <h1 class="mb-4 text-white text-xl">Top Airing</h1>
            <div v-if="topAiring != null && topAiring.length > 0"
                class="flex overflow-x-auto gap-5 w-full snap-x scroll-smooth">
                <RecentReleaseComponent v-for="ta in topAiring" :key="ta" :release="ta" class="snap-start" />
            </div>
            <div v-else class="flex overflow-x-auto gap-5 w-full snap-x scroll-smooth">
                <!-- <SpiningLoading></SpiningLoading> -->
                <LoadingAnimeCard v-for="i in 5" :key="i"></LoadingAnimeCard>
            </div>
        </div>

        <div class="w-full text-white">
            <div class="flex items-center">
                <h1 class="mb-4 text-lg">Latest Update - {{ date.month }}, {{ date.day }}</h1>
                <button class="mb-4 ml-4 p-2 bg-gray-600 rounded-xl" v-if="skipDate > 0" @click="previousDate">
                    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
                        stroke="currentColor" class="w-4 h-4">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M15.75 19.5L8.25 12l7.5-7.5" />
                    </svg>
                </button>
                <button class="mb-4 ml-4 p-2 bg-gray-600 rounded-xl" @click="nextDate">
                    <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5"
                        stroke="currentColor" class="w-4 h-4">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M8.25 4.5l7.5 7.5-7.5 7.5" />
                    </svg>
                </button>
            </div>
            <div v-if="!loading.todayStreaming && lists && lists.length > 0" class="w-full text-xs md:text-sm">
                <div v-for="l in lists" :key="l" class="mb-3 bg-gray-800 px-3 py-4 rounded-lg">
                    <nuxt-link :to="'/search/' + l.title" class="flex w-full justify-between font-bold">
                        <div class="w-9/12">
                            <h1 class="">
                                <span class="block md:inline-block mr-5 text-xs text-gray-400 mb-2 md:mb-0">
                                    {{ l.time }}
                                </span>
                                {{ l.title }}
                            </h1>
                        </div>
                        <div class="flex items-center justify-end text-right w-3/12 text-purple-400">
                            {{ l.episode }}
                        </div>
                    </nuxt-link>
                </div>
            </div>
            <div v-else class="w-full">
                <h1 class="text-gray-500 my-5 text-xs font-bold">
                    Loading Today Streaming List <span id="animateDot">...</span>
                </h1>
            </div>
        </div>
    </div>
</template>

<script>

export default {

    data() {
        return {
            isOpen: false,
            recentRelease: [],
            server: '',
            topAiring: [],
            result: [],
            lists: [],
            date: {
                year: '',
                month: '',
                day: ''
            },
            fullDate: '',
            loading: {
                todayStreaming: true,
            },
            skipDate: 0,
        }
    },
    mounted() {
        useHead({
            title: 'Home',
            meta: [
                {
                    name: 'description',
                    content: 'Anime'
                },
                {
                    name: 'keywords',
                    content: 'Anime'
                }
            ]
        })
        this.getServer()
        if (this.isOpen) {
            return
        } else {
            this.getRecentRelease()
            this.getTodayStreaming().then(() => {
                this.loading.todayStreaming = false
                setTimeout(() => {
                    this.horizonScroll()
                }, 300);
            }).catch(() => {
                this.loading.todayStreaming = false
            })
        }

        this.server = localStorage.getItem('server') ?? ''
        if (this.server == 'gogoanime') {
            this.getTopAiring()
        }

        if (this.loading.todayStreaming) {
            let animateDot = document.getElementById('animateDot')
            setInterval(() => {
                if (animateDot.innerHTML.length >= 3) {
                    animateDot.innerHTML = ''
                } else {
                    animateDot.innerHTML += '.'
                }
            }, 500)
        }
    },
    methods: {
        async getRecentRelease() {
            const config = useRuntimeConfig();
            var url = ''

            if (localStorage.getItem('server') == 'gogoanime') {
                url = config.apiUrl + 'recent-episodes'
            } else {
                url = config.apiUrl2 + 'recent-episodes'
            }

            await fetch(url)
                .then(response => response.json())
                .then(data => {
                    if (data.results.length > 0) {
                        this.recentRelease = data.results
                    } else {
                        alert(data.message)
                    }
                })
                .catch(err => {
                    alert('Something went wrong, please try again later')
                    console.log(err)
                });
        },

        getServer() {
            localStorage.getItem('server') ? this.isOpen = false : this.isOpen = true;
        },

        async getTopAiring() {
            const config = useRuntimeConfig();
            var url = ''
            url = config.apiUrl + 'top-airing'

            await fetch(url)
                .then(response => response.json())
                .then(data => {
                    // console.log(data)
                    if (data.results.length > 0) {
                        this.topAiring = data.results
                    } else {
                        alert(data.message)
                    }
                })
                .catch(err => {
                    alert('Something went wrong, please try again later')
                    // console.log(err)
                });
        },

        horizonScroll() {
            const scrollContainer = document.querySelectorAll('.scroll-smooth');
            scrollContainer.forEach((container) => {
                container.addEventListener('mouseover', (m) => {
                    container.addEventListener('wheel', (e) => {
                        var scrollAmount = (e.deltaY || e.detail || e.wheelDelta) * 10;
                        container.scrollLeft += scrollAmount;
                        e.preventDefault();
                    });
                })
            });
        },
        async getTodayStreaming(retryCount = 3, retryDelay = 1000) {
            this.lists = []
            this.loading.todayStreaming = true;

            var today = new Date();
            if (this.skipDate > 0) {
                today.setDate(today.getDate() + this.skipDate);
            } else if (this.skipDate < 0) {
                today.setDate(today.getDate() - this.skipDate);
            }
            this.fullDate = today;
            var dd = this.date.day = String(today.getDate()).padStart(2, '0');
            var mm = this.date.month = String(today.getMonth() + 1).padStart(2, '0'); //January is 0!
            var yyyy = this.date.year = today.getFullYear();

            this.date.month = this.fullDate.toLocaleString('default', { month: 'long' })
            let config = useRuntimeConfig();

            return await fetch(config.corsApi + "https://zoro.to/ajax/schedule/list?tzOffset=-480&date=" + yyyy + '-' + mm + '-' + dd)
                .then((res) => res.json()).then((html) => {
                    var parser = new DOMParser();
                    var doc = parser.parseFromString(html.html, "text/html");
                    this.result = doc.querySelectorAll(".tsl-link");
                    this.result.forEach((item, index) => {
                        // console.log("item", item);
                        if (this.lists[index] === undefined) {
                            this.lists[index] = {};
                        }
                        this.lists[index].title = item.querySelector(".dynamic-name").innerText;
                        this.lists[index].time = item.querySelector(".time").innerText;
                        this.lists[index].episode = item.querySelector(".btn-play").innerText;
                    });
                    this.loading.todayStreaming = false;
                }).catch((err) => {
                    console.log("Error: ", err);
                    if (retryCount > 0) {
                        console.log("Retrying in " + (retryDelay / 1000) + "ms...");
                        return new Promise(resolve => setTimeout(resolve, retryDelay)).then(() => {
                            this.newFetchCall(retryCount - 1, retryDelay);
                        })
                    }
                })
        },
        nextDate() {
            this.loading.todayStreaming = true;
            this.skipDate += 1;
            this.getTodayStreaming()
        },
        previousDate() {
            this.loading.todayStreaming = true;
            this.skipDate -= 1;
            this.getTodayStreaming()
        }
    }
}

</script>

<style></style>