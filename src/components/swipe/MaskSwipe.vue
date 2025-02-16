<script setup lang="ts">
import { ref, computed, onMounted, watch, CSSProperties } from 'vue';
import type { PropType } from 'vue';

type IndicatorPosition = 'left' | 'center' | 'right';
type EffectType = 'radial' | 'conic';
type IndicatorColor = {
  default: string,
  active: string,
} 

const props = defineProps({
  imgList: {
    type: Array as PropType<string[]>,
    default: () => [],
    required: true,
  },
  duration: {
    type: Number,
    default: 3,
  },
  transitionDuration: {
    type: Number,
    default: 1,
  },
  maskPositionFrom: {
    type: String,
    default: 'left',
  },
  maskPositionTo: {
    type: String,
    default: 'right',
  },
  maskImage: String,
  maskSize: String,
  effectType: {
    type: String as PropType<EffectType>,
    default: '',
  },
  showIndicator: {
    type: Boolean,
    default: true,
  },
  indicatorColor: {
    type: Object as PropType<IndicatorColor>,
    default: {
      default: "gray",
      active: "white",
    }
  },
  indicatorPosition: {
    type: String as PropType<IndicatorPosition>,
    default: 'center',
  },
});

const currentIndex = ref(0);
const oldCurrentIndex = ref(0);
const imgList = ref([...props.imgList, props.imgList[0]]);
const getInitZindex = () => {
  const arr = [1];
  for (let i = imgList.value.length - 1; i >= 1; i--) {
    arr.unshift(arr[0] + 1);
  }
  return arr;
};
const zIndexArr = ref([...getInitZindex()]);
const maskPosition = ref(props.maskPositionFrom);
const transition = ref(`all ${props.transitionDuration}s`);
const animation = ref('');

const transitionDuration = props.transitionDuration * 1000;
const duration = props.duration * 1000;

watch(currentIndex, () => {
  if (currentIndex.value === 0) {
    zIndexArr.value = [...getInitZindex()];
  }
  maskPosition.value = props.maskPositionFrom;
  animation.value = '';
  transition.value = 'none';
});
const execAnimation = () => {
  transition.value = `all ${props.transitionDuration}s`;
  maskPosition.value = props.maskPositionFrom;
  maskPosition.value = props.maskPositionTo;
  if (props.effectType === 'radial') {
    animation.value = `radial-ani ${props.transitionDuration}s linear forwards`;
  } else if (props.effectType === 'conic') {
    animation.value = `conic-ani ${props.transitionDuration}s linear forwards`;
  }
  oldCurrentIndex.value = (currentIndex.value + 1) % (imgList.value.length - 1);

  setTimeout(() => {
    zIndexArr.value[currentIndex.value] = 1;
    currentIndex.value = (currentIndex.value + 1) % (imgList.value.length - 1);
  }, transitionDuration);
};

onMounted(() => {
  const firstDelay = duration - transitionDuration;
  function animate() {
    execAnimation();
    setTimeout(animate, duration);
  }
  setTimeout(animate, firstDelay);
});

const maskImage = computed(() => {
  if (props.effectType === 'radial') {
    return `radial-gradient(circle,
    transparent calc(var(--radial-radius) - 5%),
    #fff calc(var(--radial-radius) + 5%))`;
  }
  if (props.effectType === 'conic') {
    return `conic-gradient(
                transparent 0deg,
                transparent calc(var(--angle) - 10deg),
                #fff calc(var(--angle) + 10deg),
                #fff 360deg
              )
          `;
  }
  return props.maskImage;
});
const getCurrentStyle = (index: number) => {
  if (index !== currentIndex.value) return {};
  return {
    transition: transition.value,

    'mask-image': maskImage.value,
    '-webkit-mask-image': maskImage.value,

    'mask-position': maskPosition.value,
    '-webkit-mask-position': maskPosition.value,

    'mask-size': props.maskSize,
    '-webkit-mask-size': props.maskSize,

    animation: animation.value,
  };
};
</script>

<template>
  <div class="fly-swipe-container">
    <div class="fly-swipe-item-wrap">
      <div
        class="swipe-item"
        :class="{ 'swipe-item-mask': index === currentIndex }"
        v-for="(url, index) in imgList"
        :key="index"
        :style="{ zIndex: zIndexArr[index], ...getCurrentStyle(index) }"
      >
        <img :src="url" alt="" />
      </div>
    </div>
    <div class="fly-indicator" v-show="showIndicator">
      <div
        class="fly-indicator-item"
        :class="{ 'fly-indicator-item-active': index === oldCurrentIndex }"
        v-for="(_, index) in imgList.slice(0, imgList.length - 1)"
        :key="index"
      ></div>
    </div>
  </div>
</template>

<style>
@property --radial-radius {
  syntax: '<percentage>';
  inherits: true;
  initial-value: -5%;
}

@keyframes radial-ani {
  to {
    --radial-radius: 105%;
  }
}

@property --angle {
  syntax: '<angle>';
  inherits: true;
  initial-value: -10deg;
}
@keyframes conic-ani {
  to {
    --angle: 370deg;
  }
}
</style>
<style lang="less" scoped>
.fly-swipe-container {
  position: relative;
  overflow: hidden;
  width: 100%;
  height: inherit;

  .fly-swipe-item-wrap {
    .swipe-item:first-child {
      position: relative;
    }
    .swipe-item {
      position: absolute;
      width: 100%;
      top: 0;
      left: 0;

      img {
        display: block;
        width: 100%;
        object-fit: cover;
      }
    }
    .swipe-item-mask {
      mask-repeat: no-repeat;
      -webkit-mask-repeat: no-repeat;
      mask-size: cover;
      -webkit-mask-size: cover;
      //mask-image: url('../../assets/swipe/mask4.png');
      //-webkit-mask-image: url(../../assets/swipe/mask4.png);
    }
  }

  .fly-indicator {
    display: flex;
    justify-content: v-bind('props.indicatorPosition');
    align-items: center;
    z-index: 666;
    position: absolute;
    width: 100%;
    bottom: 15px;
    padding: 0 15px;
    .fly-indicator-item {
      margin: 0 5px;
      width: 10px;
      height: 10px;
      border-radius: 50%;
      background: v-bind('props.indicatorColor.default');
    }
    .fly-indicator-item-active {
      background: v-bind('props.indicatorColor.active');
      transition: all 0.5s;
    }
  }
}
</style>
