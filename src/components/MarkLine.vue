<template>
  <div class="h-full">
    <div
      v-for="line in lines"
      :key="line"
      v-show="lineStates[line]"
      class="absolute bg-[#59c7f9] z-50"
      :class="{
        'w-full h-[1px] top-[50%] translate-y-[-50%]': isXLine(line),
        'w-[1px] h-full left-[50%] translate-x-[-50%]': !isXLine(line),
      }"
    ></div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch } from "vue";
import { useComponentStore } from "@/stores/component";
import { getCompRotatedStyle } from "@/utils/style";
import { useEditorStore } from "@/stores/editor";
import { useShapeStore } from "@/stores/shape";

const lines = ref(["lx", "ly"]);
function isXLine(line: string) {
  return line.includes("x");
}

const lineStates = ref({
  lx: false,
  ly: false,
});

const componentStore = useComponentStore();
const editorStore = useEditorStore();
const shapeStore = useShapeStore();

watch(
  () => ({
    isMove: shapeStore.isMoving,
    shapeStyle: shapeStore.hasChangeShapeStyle,
    style: componentStore.curComponent?.style,
  }),
  (newVal) => {
    if (newVal.isMove) {
      showLineAfterChangePos();
    } else {
      hiddenLine();
      if (newVal.shapeStyle) {
        showLineAfterChangeSize();
      }
    }
  },
  {
    deep: true,
  }
);

function hiddenLine() {
  Object.keys(lineStates.value).forEach((key) => {
    lineStates.value[key] = false;
  });
}

const diff = 20;
function isNearly(dragValue, targetValue) {
  return Math.abs(dragValue - targetValue) <= diff;
}

const isInCenter = ref(false);
const compStyleInCenter = ref();

function showLineAfterChangePos() {
  const curCompStyle = getCompRotatedStyle(componentStore.curComponent.style);
  const curCompHalfWidth = curCompStyle.width / 2;
  const curCompHalfHeight = curCompStyle.height / 2;

  const targetRect = editorStore.editorRect;
  const targetX = targetRect.width / 2;
  const targetY = targetRect.height / 2;

  hiddenLine();
  isInCenter.value = false;

  if (isNearly(curCompStyle.top, targetY)) {
    lineStates.value = {
      ...lineStates.value,
      lx: true,
    };
    const value = curCompStyle.rotate != 0 ? translateCurComponentPos("top", targetY, curCompStyle) : targetY;
    componentStore.setShapeSingleStyle("top", value);
  }

  if (isNearly(curCompStyle.top + curCompHalfHeight, targetY)) {
    lineStates.value = {
      ...lineStates.value,
      lx: true,
    };
    const value =
      curCompStyle.rotate != 0
        ? translateCurComponentPos("top", targetY - curCompHalfHeight, curCompStyle)
        : targetY - curCompHalfHeight;
    componentStore.setShapeSingleStyle("top", value);

    isInCenter.value = true;
    compStyleInCenter.value = {
      width: curCompStyle.width,
      height: curCompStyle.height,
    };
  }

  if (isNearly(curCompStyle.top + curCompStyle.height, targetY)) {
    lineStates.value = {
      ...lineStates.value,
      lx: true,
    };
    const value =
      curCompStyle.rotate != 0
        ? translateCurComponentPos("top", targetY - curCompStyle.height, curCompStyle)
        : targetY - curCompStyle.height;
    componentStore.setShapeSingleStyle("top", value);
  }

  if (isNearly(curCompStyle.left, targetX)) {
    lineStates.value = {
      ...lineStates.value,
      ly: true,
    };
    const value = curCompStyle.rotate != 0 ? translateCurComponentPos("left", targetX, curCompStyle) : targetX;
    componentStore.setShapeSingleStyle("left", value);
  }

  if (isNearly(curCompStyle.left + curCompHalfWidth, targetX)) {
    lineStates.value = {
      ...lineStates.value,
      ly: true,
    };
    const value =
      curCompStyle.rotate != 0
        ? translateCurComponentPos("left", targetX - curCompHalfWidth, curCompStyle)
        : targetX - curCompHalfWidth;
    componentStore.setShapeSingleStyle("left", value);

    isInCenter.value = true;
    compStyleInCenter.value = {
      width: curCompStyle.width,
      height: curCompStyle.height,
    };
  }

  if (isNearly(curCompStyle.left + curCompStyle.width, targetX)) {
    lineStates.value = {
      ...lineStates.value,
      ly: true,
    };
    const value =
      curCompStyle.rotate != 0
        ? translateCurComponentPos("left", targetX - curCompStyle.width, curCompStyle)
        : targetX - curCompStyle.width;
    componentStore.setShapeSingleStyle("left", value);
  }
}

function translateCurComponentPos(key, pos, curCompStyle) {
  const { width, height } = componentStore.curComponent.style;
  if (key === "top") {
    return Math.round(pos - (height - curCompStyle.height) / 2);
  } else if (key === "left") {
    return Math.round(pos - (width - curCompStyle.width) / 2);
  }
}

function showLineAfterChangeSize() {
  const curCompStyle = componentStore.curComponent.style;

  const targetRect = editorStore.editorRect;
  const targetHalfWidth = targetRect.width / 2;
  const targetHalfHeight = targetRect.height / 2;

  if (isInCenter.value && curCompStyle.rotate == 0) {
    if (isNearly(curCompStyle.left, targetHalfWidth)) {
      lineStates.value = {
        ...lineStates.value,
        ly: true,
      };

      componentStore.setShapeSingleStyle("left", targetHalfWidth);
      componentStore.setShapeSingleStyle("width", compStyleInCenter.value.width / 2);
    }

    if (isNearly(curCompStyle.left + curCompStyle.width, targetHalfWidth)) {
      lineStates.value = {
        ...lineStates.value,
        ly: true,
      };
      componentStore.setShapeSingleStyle("width", targetHalfWidth - curCompStyle.left);
    }

    if (isNearly(curCompStyle.top + curCompStyle.height, targetHalfHeight)) {
      lineStates.value = {
        ...lineStates.value,
        lx: true,
      };
      componentStore.setShapeSingleStyle("height", targetHalfHeight - curCompStyle.top);
    }

    if (isNearly(curCompStyle.top, targetHalfHeight)) {
      lineStates.value = {
        ...lineStates.value,
        lx: true,
      };

      componentStore.setShapeSingleStyle("top", targetHalfHeight);
      componentStore.setShapeSingleStyle("height", compStyleInCenter.value.height / 2);
    }
  }
}
</script>
