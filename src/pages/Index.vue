<template>
  <div style="width: 100%; max-width: 480px; margin: 10px auto;">
    <q-card>
      <q-card-media>
        <img src="../assets/hero.jpg">
      </q-card-media>
      <q-card-main>
        <q-field
          icon="stars"
          :label="`当前等级：${level}级 ${sStar}星 ${rank}`"
          orientation="vertical"
          inset="icon"
        >
          <q-slider v-if="!emulating" v-model="sLevel" :min="6" :max="25" :step="1" snap markers />
          <q-rating v-if="!emulating" v-model="sStar" color="amber-8" size="24px" />
        </q-field>

        <q-field
          icon="my_location"
          :label="`目标等级：${target}级 ${reward}`"
          orientation="vertical"
          inset="icon"
          class="q-mt-lg"
        >
          <q-slider v-if="!emulating" v-model="sTarget" :min="5" :max="20" :step="5" snap markers />
        </q-field>

        <q-field
          icon="network_check"
          :label="`卡组胜率：${sRate}%`"
          orientation="vertical"
          inset="icon"
          class="q-mt-lg"
        >
          <q-slider v-if="!emulating" v-model="sRate" :min="40" :max="80" />
        </q-field>

        <q-card-separator v-if="emulating" class="q-mt-lg"/>

        <q-tabs v-if="emulating" animated swipeable align="justify" inverted color="primary" no-pane-border>
          <q-tab slot="title" label="悲观" name="tab-1" icon="sentiment_very_dissatisfied" />
          <q-tab slot="title" label="保守" name="tab-2" icon="sentiment_dissatisfied" />
          <q-tab slot="title" label="中立" name="tab-3" icon="face" default />
          <q-tab slot="title" label="开放" name="tab-4" icon="sentiment_satisfied" />
          <q-tab slot="title" label="乐观" name="tab-5" icon="sentiment_satisfied_alt" />

          <q-tab-pane v-for="(v, i) in showData" :key="i" :name="`tab-${i + 1}`">
            <q-list no-border dense>
              <q-item>
                <q-item-main label="对局次数" />
                <q-item-side right>{{ v.play }}局</q-item-side>
              </q-item>
              <q-item>
                <q-item-main label="获胜次数" />
                <q-item-side right>{{ v.win }}局</q-item-side>
              </q-item>
              <q-item>
                <q-item-main label="连胜奖励" />
                <q-item-side right>{{ v.bonus }}星</q-item-side>
              </q-item>
              <q-item>
                <q-item-main label="金币奖励" />
                <q-item-side right>{{ v.coin }}金</q-item-side>
              </q-item>
            </q-list>
          </q-tab-pane>
        </q-tabs>
        <q-btn v-if="emulating" @click="reset" label="重置" class="full-width" :loading="running" :percentage="percentage" dark-percentage/>
      </q-card-main>
      <q-card-actions>
        <q-btn v-if="!emulating" color="primary" @click="emu" label="模拟" class="full-width"/>
      </q-card-actions>
    </q-card>
  </div>
</template>

<style>
</style>

<script>
const levels = [];
for (let i = 5; i <= 25; i += 1) {
  levels.push(i);
}
const rates = [];
for (let i = 30; i <= 80; i += 1) {
  rates.push(i);
}
const rewards = {
  '20': '海巨人金质宝箱',
  '15': '食人魔法师银质宝箱',
  '10': '守护着铜质宝箱',
  '5': '持盾卫士木质宝箱',
};

const ranks = [
  '愤怒的小鸡',
  '麻疯侏儒',
  '银色侍从',
  '鱼人袭击者',
  '南海船工',
  '持盾卫士',
  '工程师学徒',
  '巫师学徒',
  '牛头人战士',
  '任务达人',
  '银月城卫兵',
  '团队领袖',
  '恐怖海盗',
  '战歌指挥官',
  '王牌猎人',
  '食人魔法师',
  '白银之手骑士',
  '霜狼督军',
  '烈日行者',
  '战争古树',
]

const isSafe = step => step >= 100 || step % 25 === 0;

const emulate = (from, to, rate) => {
  let now = from;
  let last2 = '00';
  let play = 0;
  let win = 0;
  let bonus = 0;
  while (now > to) {
    play += 1;
    if (Math.random() * 100 < rate) {
      win += 1;
      now -= 1;
      if (last2 === '11') {
        now -= 1;
        bonus += 1;
      }
      last2 = `${last2[1]}1`;
    } else {
      if (!isSafe(now)) {
        now += 1;
      }
      last2 = `${last2[1]}0`;
    }
  }
  return {
    play,
    win,
    bonus,
    coin: Math.floor(win / 3) * 10,
  };
};

const batch = (from, to, rate, times, tfn, ffn) => {
  const results = [];
  let i = 0;
  const run = () => {
    if (i == times) {
      return ffn();
    }
    i += 1;
    results.push(emulate(from, to, rate));
    results.sort((a, b) => a.play - b.play);
    tfn([
      results[i - 1],
      results[Math.floor(i / 4 * 3)],
      results[Math.floor(i / 2)],
      results[Math.floor(i / 4)],
      results[0],
    ], i, results);
    setTimeout(run, 100);
  };
  run();
};

export default {
  name: 'PageIndex',
  data: () => ({
    levels,
    rates,
    sLevel: 6,
    sStar: 0,
    sTarget: Number,
    sRate: Number,
    time: 0,
    times: 99,
    running: false,
    emulating: false,
  }),
  created: function created() {
    this.sTarget = 5;
    this.sRate = 50;
  },
  computed: {
    reward: function () {
      return rewards[this.sTarget];
    },
    rank: function () {
      return ranks[this.sLevel - 6];
    },
    level: function () {
      return 31 - this.sLevel;
    },
    target: function () {
      return 25 - this.sTarget;
    },
    percentage: function () {
      return Math.round(this.time / this.times * 100);
    }
  },
  methods: {
    emu: function emu() {
      this.running = true;
      this.emulating = true;
      const from = (this.level * 5) - this.sStar;
      const to = this.target * 5;
      batch(from, to, this.sRate, this.times, (showData, time, all) => {
        this.showData = showData;
        this.time = time;
        this.$forceUpdate();
      }, () => {
        this.running = false;
      });
    },
    reset: function reset() {
      this.emulating = false;
    }
  },
};
</script>
