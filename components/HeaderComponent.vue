<script setup lang="ts">

</script>

<template>
  <div class="header-wrapper">
    <div class="header-container">
      <div><a href="#about">" About "</a></div>
      <div><a href="mailto:drivakosv@gmail.com">" Contact "</a></div>
      <div><a target="_blank" href="https://www.linkedin.com/in/evangelos-drivakos-170b0621b/">" LinkedIn "</a></div>
    </div>
    <h1 @mouseover="startAnimation">{{ text }}</h1>
  </div>
</template>


<script>
export default {
  data() {
    return {
      text: "Drivakos",
      letters: "ABCDEFGHIJKLMNOPQRSTUVWXYZ",
      interval: null,
      isAnimating: false,
    };
  },
  methods: {
    startAnimation(event) {
      if (this.isAnimating) return;

      let iteration = 0;
      const originalText = this.text;
      this.isAnimating = true;

      clearInterval(this.interval);

      this.interval = setInterval(() => {
        this.text = this.text
            .split("")
            .map((letter, index) => {
              if (index < iteration) {
                return originalText[index];
              }

              return this.letters[Math.floor(Math.random() * 26)];
            })
            .join("");

        if (iteration >= originalText.length) {
          clearInterval(this.interval);
          this.text = originalText;
          this.isAnimating = false;
        }

        iteration += 1 / 3;
      }, 30);
    },
  },
};
</script>
