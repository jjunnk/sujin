<template>
  <v-card
    id="FastestGameWrapper"
    flat
    height="100%"
    :color="$vuetify.theme.dark ? 'base1' : 'info'"
  >
    <v-container class="ma-auto">
      <v-btn id="clickbtn" @click="updateCount"></v-btn>
      <h1 id="status">{{ status }}</h1>
      <h1>
        You clicked <b>{{ numClicks }}</b> times
      </h1>
      <h1>
        You have <b>{{ secs }} secs</b> left
      </h1>
    </v-container>
  </v-card>
</template>

<script type="text/javascript">
export default {
  data() {
    return {
      active: false,
      numClicks: 0,
      secs: 10,
      colors: ["#F7A381", "#EBC0C0", "#C0EBCC", "#CCCFEB", "#C0EBCC"],
      status: "",
    };
  },
  created() {
    setInterval(() => {
      if (this.active) {
        this.secs--; // 1 감소
        if (this.secs == 0) {
          this.active = false;
          const btn = document.getElementById("clickbtn");
          btn.classList.add("over");
          this.status = "Game Over";
        }
      }
    }, 1000);
  },
  methods: {
    updateCount() {
      this.numClicks += 1;
      const btn = document.getElementById("clickbtn");
      const color = this.colors[Math.round(Math.random() * (this.colors.length - 1))];
      btn.style.backgroundColor = color;

      if (this.active == false) {
        this.secs = 10;
        this.numClicks = 0;
        this.active = true;
      }

      if (this.numClicks % 10 == 0) {
        btn.style.width = btn.offsetWidth - 15 + "px";
        btn.style.height = btn.offsetHeight - 15 + "px";
      }
    },
  },
};
</script>

<style scoped lang="scss">
@import "@/assets/scss/_variables.scss";

.container {
  position: absolute;
  transform: translate(-50%, -40%);
  top: 40%;
  left: 50%;
  text-align: center !important;
}

#clickbtn {
  position: relative;
  border: none;
  color: #fff;
  width: 200px;
  height: 200px;
  transition: 0.3s;
  border-radius: 50%;
  background-color: #f7a381;
  margin-bottom: 1.7em;
  transition: all 0.4s;
  transition-timing-function: ease-in-out;
}

#clickbtn:active {
  width: 190px;
  height: 190px;
}

#clickbtn:focus {
  outline: none;
}

#status {
  background: transparent;
  position: relative;
  font-size: 60px;
  text-transform: uppercase;
  font-weight: bold;
}

#timer {
  text-align: center;
  margin-top: 140px;
}

h1 {
  color: #fff;
  font-weight: 400;
  font-family: $fontfamily;
  font-size: 1.8em;
}

.over {
  background-color: transparent;
  display: none;
}
</style>
