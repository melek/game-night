<template>
  <header>
    <nav>
      <h1>
        <a href="/">Game Night</a>
        <transition appear name="fade">
              <slot></slot>
        </transition>
      </h1>      
      <input type="checkbox" id="menu-toggle">
      <label for="menu-toggle">&#9776;</label>
			<a href="schedule">Schedule</a>
			<a href="games">Games</a>
      <a href="leaderboard">Leaderboard</a>
    </nav>
  </header>
</template>

<script>
export default {
    
}
</script>

<style scoped>
.fade-enter {
  opacity: 0;
}

.fade-enter-active {
  transition: opacity 1s;
}

.fade-leave-active {
  transition: opacity 1s;
  opacity: 0;
}
</style>
