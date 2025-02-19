<template>
  <div class="carousel-wrapper">
    <div class="carousel-container">

      <button class="carousel__button prev" aria-label="Previous slide" @click="prevSlide">&#9001;</button>

      <div class="carousel" @transitionend="handleTransitionEnd">
        <div class="carousel__inner" :style="carouselStyle">
          <div v-for="(image, index) in cloneData"
               :key="index"
               :style="carouselItemStyle"
               class="carousel__item"
               @click="toggleImageSelection(image)"
          >
            <img :src="image.download_url"
                 :alt="image.author"
                 loading="lazy"
                 :class="{'selected': isImageSelected(image)}"
            />
          </div>
        </div>
      </div>

      <button class="carousel__button next" aria-label="Next slide" @click="nextSlide">&#9002;</button>
    </div>

    <div class="carousel__indicators">
      <span
          v-for="(image, index) in data"
          :key="index"
          :class="['indicator', { active: isActive(index) }]"
          @click="goToSlide(index)"
      ></span>
    </div>

    <div class="selected-images">
      <h3>Selected Images</h3>
      <ul>
        <li v-for="(image, index) in selectedImages" :key="index">
          <img :src="image.download_url" :alt="image.author" class="thumbnail" />
          <a :href="image.url" target="_blank" :aria-label="`Open image by ${image.author}`">{{ image.download_url }}</a>
        </li>
      </ul>
    </div>

  </div>
</template>

<script>
export default {
  name: 'AppImageCarousel',
  props: {
    data: {
      type: Array,
      required: true,
    },
  },
  data() {
    return {
      currentIndex: 0,
      visibleItems: 1,
      transitioning: false,
      selectedImages: [],
    };
  },

  computed: {
    cloneData() {
      return [...this.data, ...this.data, ...this.data];
    },
    carouselStyle() {
      return {
        transform: `translateX(-${this.currentIndex * (100 / this.visibleItems)}%)`,
        transition: this.transitioning ? 'transform 0.1s linear' : 'none',
      };
    },
    carouselItemStyle() {
      return {
        width: `${100 / this.visibleItems}%`,
        transition: this.transitioning ? 'width 0.3s linear' : 'none',
      };
    },
  },

  created() {
    this.updateVisibleItems();
    window.addEventListener('resize', this.updateVisibleItems);
    this.currentIndex = this.data.length;
  },

  methods: {
    nextSlide() {
      if (this.transitioning) return;
      this.transitioning = true;
      this.currentIndex++;
    },

    prevSlide() {
      if (this.transitioning) return;
      this.transitioning = true;
      this.currentIndex--;
    },

    handleTransitionEnd() {
      if (this.currentIndex >= this.data.length * 2) {
        this.currentIndex = this.data.length;
      } else if (this.currentIndex < this.data.length) {
        this.currentIndex = this.data.length * 2 - 1;
      }
      this.transitioning = false;
    },

    updateVisibleItems() {
      const width = window.innerWidth;
      const itemWidth = 300;
      let visibleItems = Math.floor(width / itemWidth);
      if (visibleItems < 1) visibleItems = 1;
      this.visibleItems = visibleItems;
    },

    isActive(index) {
      return index === Math.floor(this.currentIndex % this.data.length);
    },

    goToSlide(index) {
      this.transitioning = true;
      this.currentIndex = index + this.data.length;

      setTimeout(() => {
        this.transitioning = false;
      }, 300);
    },

    toggleImageSelection(image) {
      const index = this.selectedImages.findIndex((img) => img.id === image.id);
      if (index === -1) {
        this.selectedImages.push(image);
      } else {
        this.selectedImages.splice(index, 1);
      }
    },

    isImageSelected(image) {
      return this.selectedImages.some((img) => img.id === image.id);
    },

  },

  beforeUnmount() {
    window.removeEventListener('resize', this.updateVisibleItems);
  },
};
</script>

<style lang="scss" scoped>

.carousel-container {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.carousel {
  overflow: hidden;
  width: 90%;

  &__inner {
    display: flex;
    transition: transform 0.5s ease-in-out;
  }

  &__item {
    flex: 0 0 auto;
    cursor: pointer;
    padding: 5px;
    box-sizing: border-box;

    img {
      width: 100%;
      min-height: 220px;
      object-fit: contain;
      transition: opacity 0.3s ease, transform 0.3s ease;

      &:hover {
        opacity: 0.7;
      }
    }

    .selected {
      transform: scale(0.97);
      transition: transform 0.3s ease-in-out;
    }
  }

  &__button {
    background: transparent;
    border: none;
    font-size: 24px;
    cursor: pointer;
    color: #333;

    &:hover {
      color: #000000;
    }

    &:active {
      transform: scale(0.9);
    }
  }

  &__indicators {
    display: flex;
    justify-content: center;
    margin-top: 10px;
  }
}

.indicator {
  width: 12px;
  height: 12px;
  margin: 0 6px;
  border-radius: 50%;
  background-color: transparent;
  border: 2px solid rgba(0, 123, 255, 0.5);
  transition: background-color 0.3s ease, border-color 0.3s ease, transform 0.3s ease;
  cursor: pointer;
  position: relative;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);

  &:hover {
    background: rgba(0, 123, 255, 0.3);
    border-color: rgba(0, 123, 255, 0.8);
    transform: scale(1.2);
  }

  &::before {
    content: "";
    position: absolute;
    top: 50%;
    left: 50%;
    width: 6px;
    height: 6px;
    background: rgba(0, 123, 255, 0.7);
    border-radius: 50%;
    transform: translate(-50%, -50%) scale(0);
    transition: transform 0.3s ease-in-out;
  }
}

.active {
  background-color: rgba(0, 123, 255, 0.7);
  border-color: rgba(0, 123, 255, 1);
  transform: scale(1.2);

  &::before {
    transform: translate(-50%, -50%) scale(1);
  }
}

.selected-images {
  margin-top: 20px;
  background: linear-gradient(135deg, #f3f4f6, #ffffff);
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  h3 {
    font-size: 1.6rem;
    font-weight: 700;
    margin-bottom: 15px;
    color: #222;
    text-align: center;
    position: relative;

    &::after {
      content: "";
      display: block;
      width: 50px;
      height: 3px;
      background: #007bff;
      margin: 8px auto 0;
      border-radius: 2px;
    }
  }

  ul {
    list-style: none;
    padding: 0;
  }

  li {
    display: flex;
    align-items: center;
    background: #fff;
    border-radius: 10px;
    padding: 12px;
    margin-bottom: 12px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
    transition: all 0.3s ease;
    position: relative;

    &:hover {
      background: #f0f8ff;
      transform: translateX(8px);
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.12);
    }

    &::before {
      content: "✔";
      position: absolute;
      left: -12px;
      top: 50%;
      transform: translateY(-50%);
      background: green;
      color: white;
      font-size: 14px;
      width: 20px;
      height: 20px;
      display: flex;
      align-items: center;
      justify-content: center;
      border-radius: 50%;
      opacity: 0;
      transition: opacity 0.3s ease-in-out;
    }

    &:hover::before {
      opacity: 1;
    }

    & img {
      width: 60px;
      height: 60px;
      object-fit: cover;
      border-radius: 10px;
      margin-right: 15px;
      box-shadow: 0 3px 6px rgba(0, 0, 0, 0.1);
      transition: transform 0.3s ease;
    }

    & p {
      font-size: 1rem;
      color: #444;
      margin: 0;
      flex: 1;
    }

    & a {
      background: #007bff;
      color: white;
      padding: 6px 12px;
      border-radius: 6px;
      font-size: 0.9rem;
      text-decoration: none;
      transition: background 0.3s ease, transform 0.3s ease;

    }
  }
}

</style>
