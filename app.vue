<template>
  <div>
    <NuxtLayout>
      <AppHeader v-if="shouldBeDisplayed()" @toggle-side-nav="toggleSideNav" />
      <SideNav
        v-if="shouldBeDisplayed()"
        ref="sideNav"
        :class="{ 'mobile-active': isSideNavOpen }"
      />
      <div v-if="isSideNavOpen" class="overlay" @click="toggleSideNav"></div>
      <NuxtPage class="page" :class="{ shifted: isSideNavOpen }" />
    </NuxtLayout>
  </div>
</template>

<script lang="ts" setup>
import "bootstrap/dist/css/bootstrap.min.css";
import { ref } from "vue";
const route = useRoute();

const isSideNavOpen = ref(false);
const sideNav = ref(null);

const shouldBeDisplayed = () => {
  return (
    route.name !== "auth" &&
    route.name !== "register" &&
    route.name !== "verify"
  );
};

const toggleSideNav = () => {
  isSideNavOpen.value = !isSideNavOpen.value;
};

definePageMeta({
  middleware: "auth",
});
</script>

<style lang="scss">
@use "~/styles/index.scss";

.page {
  margin-left: calc(20% + 10px);
  position: absolute;
  top: 90px;
  width: calc(80% - 20px);
  background-color: #fff;
  border-radius: 5px;
  padding: 16px;
  @media screen and (max-width: 500px) {
    border: 2px solid black;
    // margin-left: 40px;
    width: 100%;
    margin-inline: auto;
    display: flex;
    flex-direction: column;
    padding: 10px;
    right: 0px;
  }
}

.overlay {
  display: none;

  @media screen and (max-width: 500px) {
    display: block;
    position: fixed;
    bottom: 0;
    right: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.2);
    z-index: 9;
    // overflow-y: hidden;
  }
}

aside {
  @media screen and (max-width: 500px) {
    position: fixed;
    top: 0;
    left: 0;
    height: 100vh;
    width: 250px;
    z-index: 1000;
    transform: translateX(-100%);
    transition: transform 0.3s ease;
    background-color: #260021;

    &.mobile-active {
      transform: translateX(0);
      display: block !important;
      z-index: 100;
      // overflow: hidden;
    }
  }
}
</style>

<!-- <template>
  <div>
    <NuxtLayout>
      <AppHeader v-if="shouldBeDisplayed()" />
      <SideNav v-if="shouldBeDisplayed()" />
      <NuxtPage class="page" />
    </NuxtLayout>
  </div>
</template>

<script lang="ts" setup>
import "bootstrap/dist/css/bootstrap.min.css";
const route = useRoute();

const shouldBeDisplayed = () => {
  return (
    route.name !== "auth" &&
    route.name !== "register" &&
    route.name !== "verify"
  );
};

definePageMeta({
  middleware: "auth",
});
</script>

<style lang="scss">
@use "~/styles/index.scss";
.page {
  margin-left: calc(20% + 10px);
  position: absolute;
  top: 90px;
  width: calc(80% - 20px);
  background-color: #fff;
  border-radius: 5px;
  padding: 16px;
  @media screen and (max-width: 500px) {
    width: 100%;
    margin-inline: auto;
    display: flex;
    flex-direction: column;
    padding: 10px;
  }
}
</style> -->

<style lang="scss">
@use "~/styles/index.scss";
.page {
  margin-left: calc(20% + 10px);
  position: absolute;
  top: 90px;
  width: calc(80% - 20px);
  background-color: #fff;
  border-radius: 5px;
  padding: 16px;
  @media screen and (max-width: 500px) {
    border: 2px solid black;
    // margin-left: 40px;
    width: 100%;
    margin-inline: auto;
    display: flex;
    flex-direction: column;
    padding: 10px;
    right: 0px;
  }
}
</style>

<!-- <template>
  <div>
    <NuxtLayout>
      <AppHeader v-if="shouldBeDisplayed()" />
      <SideNav v-if="shouldBeDisplayed()" />
      <NuxtPage class="page" />
    </NuxtLayout>
  </div>
</template>

<script lang="ts" setup>
import "bootstrap/dist/css/bootstrap.min.css";
const route = useRoute();

const shouldBeDisplayed = () => {
  return (
    route.name !== "auth" &&
    route.name !== "register" &&
    route.name !== "verify"
  );
};

definePageMeta({
  middleware: "auth",
});
</script>

<style lang="scss">
@use "~/styles/index.scss";
.page {
  margin-left: calc(20% + 10px);
  position: absolute;
  top: 90px;
  width: calc(80% - 20px);
  background-color: #fff;
  border-radius: 5px;
  padding: 16px;
  @media screen and (max-width: 500px) {
    width: 100%;
    margin-inline: auto;
    display: flex;
    flex-direction: column;
    padding: 10px;
  }
}
</style> -->
