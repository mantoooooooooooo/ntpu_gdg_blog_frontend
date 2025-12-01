<template>
  <div class="w-full side-padding">
    <section class="hero section">
      <img loading="lazy" src="/assets/svg/orbit.svg" alt="orbit animation" />
      <div>
        <h1 class="big-heading">我們是...</h1>
        <h2 class="medium-heading">Google 學生開發者社群<br />國立臺北大學</h2>
        <p>
          成立於 2020 年，由 Google 支持的學生社群計畫，<br />
          旨在能鼓勵學生們學習與應用 Google 技術，<br />
          並提供成員們資源協助及與專家互動交流的機會。<br />
          我們致力於打造一個文組人入門的科技社群，<br />
          提供良好的學習環境，給予開發者交流的技術平台。
        </p>
      </div>
    </section>

    <section class="about section">
      <h1 class="big-heading">我們在做...</h1>
      <div class="about-container">
        <div class="about-grid">
          <div
            v-for="(item, index) in aboutItems"
            :key="index"
            class="about-grid-item box-design"
          >
            <span class="medium-heading font-bold text-center">
              {{ item.title }}
            </span>
            <NuxtLink :to="item.link">更多內容...</NuxtLink>
          </div>
        </div>

        <div class="event-info box-design">
          <h2 class="medium-heading font-bold text-center mb-4">重要活動</h2>
          <EventCard
            v-if="event"
            :event="event"
            @selected="openModal"
          ></EventCard>
          <p v-else class="medium-heading m-auto text-neutral-400 italic">
            近期無活動
          </p>
          <EventModal
            v-if="selectedEvent"
            :event="selectedEvent"
            @close="selectedEvent = null"
          ></EventModal>
        </div>
      </div>
    </section>

    <section class="articles section">
      <h1 class="big-heading">最新文章</h1>
      <NuxtLink
        :to="`/articles/${article.id}`"
        class="new-article box-design"
        v-for="article in topTwoArticles"
        :key="article.title"
      >
        <span class="line-decoration self-end"></span>
        <div class="new-article-content">
          <div class="w-[65%]">
            <p class="medium-heading">{{ article.title }}</p>
            <p class="mt-4">{{ article.description }}</p>
          </div>
          <div class="relative">
            <img
              class="rounded-full"
              :src="article.authorImage || '/img/profile.png'"
              alt="speaker-image"
            />
            <span class="absolute w-full text-center bottom-[-2rem]">
              {{ article.author }}
            </span>
          </div>
        </div>
        <span class="line-decoration"></span>
      </NuxtLink>
    </section>

    <section class="team section">
      <h1 class="big-heading">開發團隊</h1>
      <div class="w-full flex my-12 gap-4 justify-center items-center">
        <button
          class="navigation"
          @click="ScrollLeft()"
          aria-label="Scroll left"
        >
          &lt;
        </button>
        <div id="team" class="team-container no-scrollbar p-2">
          <div
            class="sub-team"
            v-for="subteam in subteams"
            :key="subteam.title"
          >
            <p class="medium-heading font-bold">{{ subteam.title }}</p>
            <div
              class="flex flex-col py-4 w-[136px] h-[330px] justify-center gap-4 items-center rounded-[--border-radius] border-2 border-[#555555] my-4"
            >
              <MemberBasic
                v-for="member in subteam.members"
                :key="member.name"
                :member="member"
              ></MemberBasic>
            </div>
          </div>
        </div>
        <button
          class="navigation"
          @click="ScrollRight()"
          aria-label="Scroll right"
        >
          &gt;
        </button>
      </div>
    </section>
  </div>
</template>

<script setup lang="ts">
  /*===== SEO Optimization - Use Nuxt's composable for Head Management =====*/
useHead({
  title: 'Google 學生開發者社群 (GDSC) 國立臺北大學 | 文組友善技術交流平台',
  meta: [
    {
      name: 'description',
      content: '成立於 2020 年，Google 學生開發者社群 國立臺北大學 (GDSC NTPU) 旨在鼓勵學生學習與應用 Google 技術，提供良好的文組入門科技學習環境。',
    },
    // 您可以根據需要添加其他 meta 標籤，例如 keywords 或 open graph
    {
      name: 'keywords',
      content: 'Google 學生開發者社群, 國立臺北大學, GDSC, GDSC NTPU, 程式設計, 專案實作, 技術交流',
    },
  ],
});
/*======================================================================*/
/*===== Get newest articles =====*/
interface Article {
  id: string;
  title: string;
  time: Date;
  author: string;
  authorImage: string;
  description: string;
}

const { data: articles } = await useFetch<Article[]>('/api/article/all');

const topTwoArticles = computed(() => {
  if (!articles.value) return [];

  return [...articles.value]
    .sort((a, b) => {
      return new Date(b.time).getTime() - new Date(a.time).getTime();
    })
    .slice(0, 2);
});
/*
const topTwoArticles = computed(() => {
  if (!articles.value) return [];
  return articles.value
    .reduce((acc, curr) => {
      if (acc.length < 2) acc.push(curr);
      // Always keep the two most recent articles
      acc.sort((a, b) => new Date(b.time) - new Date(a.time)); 
      return acc;
    }, []);
});
*/

/*===== Get upcoming event =====*/
interface Activity {
  thumbnail: string;
  title: string;
  date: Date;
  description: string;
  tags: Array<string>;
}
const today = new Date().toISOString().slice(0, 11);
const { data: events, error } = await useFetch<Activity[]>(
  `/api/activity?startDate=${today}00:00:00Z&endDate=2025-12-31T23:59:59Z&num=1`
);

const event = computed(() => events.value?.[0] || null);

const selectedEvent = ref<Activity | null>(null);

const openModal = (event: Activity) => {
  selectedEvent.value = event;
};

/*===== What we do section =====*/
const aboutItems = [
  { title: '專案實作', link: '/projects' },
  { title: '程式設計', link: '/articles' },
  { title: '課程講座', link: '/articles' },
];

/*===== Dev team element scrolling controller =====*/
function ScrollRight(): void {
  document.getElementById('team')?.scrollBy(140, 0);
}
function ScrollLeft(): void {
  document.getElementById('team')?.scrollBy(-140, 0);
}

/*===== Dev team data =====*/
const subteams = [
  {
    title: 'PM',
    members: [
      {
        name: '許甄珆',
        profile:
          'https://res.cloudinary.com/gdg-ntpu/image/upload/v1746864376/member-1_c42mar.png',
      },
      {
        name: '施尚丞',
        profile:
          'https://res.cloudinary.com/gdg-ntpu/image/upload/v1746864376/member-2_zqjr3z.png',
      },
    ],
  },
  {
    title: '行銷',
    members: [
      {
        name: '黃意捷',
        profile:
          'https://res.cloudinary.com/gdg-ntpu/image/upload/v1746864376/member-3_rrk7fp.png',
      },
      {
        name: '楊芷捷',
        profile:
          'https://res.cloudinary.com/gdg-ntpu/image/upload/v1746864377/member-4_ehskfp.png',
      },
    ],
  },
  {
    title: '前端',
    members: [
      {
        name: '陳宥任',
        profile:
          'https://res.cloudinary.com/gdg-ntpu/image/upload/v1746864377/member-5_uzdsrn.png',
      },
      {
        name: '魏琦蓁',
        profile:
          'https://res.cloudinary.com/gdg-ntpu/image/upload/v1746864377/member-6_xh2o3n.png',
      },
    ],
  },
  {
    title: '後端',
    members: [
      {
        name: '劉晉嘉',
        profile:
          'https://res.cloudinary.com/gdg-ntpu/image/upload/v1746866625/member-7_g8gyqk.png',
      },
    ],
  },
  {
    title: 'UIUX',
    members: [
      {
        name: '李芸瑄',
        profile:
          'https://res.cloudinary.com/gdg-ntpu/image/upload/v1746864376/member-9_axbpft.png',
      },
      {
        name: '余沁恩',
        profile:
          'https://res.cloudinary.com/gdg-ntpu/image/upload/v1746864376/member-10_wucf9l.png',
      },
    ],
  },
];
</script>

<style scoped>
.hero {
  gap: 3rem;
}
.big-heading {
  font-weight: bold;
}
.hero p {
  margin-top: 1.5rem;
  text-wrap: nowrap;
}

.about {
  flex-direction: column;
}
.about-container {
  display: flex;
  gap: 5rem;
  flex-grow: 1;
  margin: 2rem auto;
  justify-content: center;
}
.about-grid {
  display: grid;
  grid-template-rows: repeat(3, 1fr);
  gap: 2rem;
}
.about-grid-item {
  display: flex;
  flex-direction: column;
  color: white;
  background-color: black;
  justify-content: center;
  position: relative;
  width: 300px;
  height: 120px;
}
.about-grid-item:nth-child(2) {
  background-color: white;
  color: black;
}
.about-grid-item a {
  position: absolute;
  text-decoration-line: underline;
  bottom: 0.5rem;
  right: 2rem;
}
.about-grid-item a:hover {
  transform: scale(1.05);
}
.event-info {
  padding: 2rem 1rem;
  display: flex;
  flex-direction: column;
  width: 350px;
}

.articles {
  flex-direction: column;
  width: 90%;
  max-width: 900px;
  margin: 3rem auto;
}
.new-article {
  box-shadow: 0 0.5rem black;
  margin: 1rem 0;
  padding: 10px;
  display: flex;
  flex-direction: column;
  width: 100%;
  min-width: 250px;
}
.line-decoration {
  width: 60%;
  border: 0.5px grey solid;
  margin: 20px;
}
.new-article-content {
  width: 90%;
  display: flex;
  align-items: center;
  margin: 0 auto;
  justify-content: space-around;
}
.new-article-content img {
  width: 8rem;
  height: 8rem;
}

.team {
  flex-direction: column;
}
.navigation {
  display: none;
}
.team-container {
  display: flex;
  text-align: center;
  overflow-x: scroll;
  scroll-snap-type: x mandatory;
  scroll-behavior: smooth;
  gap: 2rem;
}
.sub-team {
  scroll-snap-align: center;
}

@media screen and (max-width: 1078px) {
  .navigation {
    display: block;
  }
}
@media screen and (max-width: 1000px) {
  .hero {
    gap: 1rem;
  }
  .hero img {
    width: 400px;
  }
  .about-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 100%;
  }
  .about-grid-item span {
    font-size: 1.5rem;
  }
  .about-grid-item a {
    font-size: 0.75rem;
  }
  .new-article-content img {
    width: 6rem;
    height: 6rem;
  }
  .new-article-content span {
    font-size: 0.9rem;
  }
}
@media screen and (max-width: 768px) {
  .hero {
    flex-direction: column;
    gap: 0;
  }
  .hero img {
    width: auto;
  }
  .new-article-content img {
    width: 4rem;
    height: 4rem;
  }
  .new-article-content span {
    font-size: 0.8rem;
  }
}
@media screen and (max-width: 475px) {
  .team-container {
    width: 150px;
    overflow-x: hidden;
  }
  .event-info {
    width: 300px;
  }
}
</style>
