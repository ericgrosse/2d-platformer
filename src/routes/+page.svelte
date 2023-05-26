<script>
  import { onMount } from 'svelte';

  // Player object
  let player = {
    x: 50, // Initial player x position
    y: 300, // Initial player y position
    width: 50,
    height: 50,
    jumping: false,
    yVelocity: 0,
    gravity: 1,
    speed: 0, // Player's current horizontal speed
    acceleration: 0.5, // Acceleration rate
    maxSpeed: 5, // Maximum horizontal speed
  };

  // Platform objects
  let platforms = [
    { x: 0, y: 350, width: 150, height: 10 }, // Starting platform
    { x: 200, y: 250, width: 100, height: 10 },
    { x: 400, y: 200, width: 150, height: 10 },
    { x: 600, y: 150, width: 100, height: 10 },
    { x: 800, y: 100, width: 150, height: 10 }, // Ending platform
  ];

  let arrowKeys = new Set();

  // Key event listeners for player movement
  function handleKeyDown(event) {
    arrowKeys.add(event.key);

    if (arrowKeys.has('ArrowLeft')) {
      player.speed = Math.max(-player.maxSpeed, player.speed - player.acceleration);
    }

    if (arrowKeys.has('ArrowRight')) {
      player.speed = Math.min(player.maxSpeed, player.speed + player.acceleration);
    }

    if (arrowKeys.has(' ')) {
      if (!player.jumping) {
        player.yVelocity = -15; // Jump velocity
        player.jumping = true;
      }
    }
  }

  function handleKeyUp(event) {
    arrowKeys.delete(event.key);

    if (!arrowKeys.has('ArrowLeft') && !arrowKeys.has('ArrowRight')) {
      player.speed = 0;
    }

    if (arrowKeys.has(' ')) {
      // Jump key still held down, don't reset speed
      arrowKeys.add('ArrowLeft');
    }
  }

  // Update player position and check for collisions
  function update() {
    player.yVelocity += player.gravity;
    player.y += player.yVelocity;

    // Update player's x position based on speed
    player.x += player.speed;

    // Apply friction to gradually reduce speed when no arrow keys are pressed
    if (!arrowKeys.has('ArrowLeft') && !arrowKeys.has('ArrowRight')) {
      if (player.speed > 0) {
        player.speed = Math.max(0, player.speed - player.acceleration);
      } else if (player.speed < 0) {
        player.speed = Math.min(0, player.speed + player.acceleration);
      }
    }

    // Check if player's base makes contact with the bottom of the browser screen
    if (player.y + player.height >= window.innerHeight) {
      player.x = 50; // Reset player's x position
      player.y = 300; // Reset player's y position
      player.yVelocity = 0; // Reset player's y velocity
      player.jumping = false; // Reset jumping flag
    }

    platforms.forEach(platform => {
      if (
        player.x < platform.x + platform.width &&
        player.x + player.width > platform.x &&
        player.y + player.height > platform.y &&
        player.y < platform.y + platform.height
      ) {
        // Collision detected, stop falling and set player position on top of the platform
        player.yVelocity = 0;
        player.y = platform.y - player.height;
        player.jumping = false;
      }
    });
  }

  onMount(() => {
    // Attach event listeners for keydown and keyup events
    window.addEventListener('keydown', handleKeyDown);
    window.addEventListener('keyup', handleKeyUp);

    // Game loop
    const gameLoop = setInterval(() => {
      update();
    }, 16); // Run update every 16ms (roughly 60 FPS)

    // Clean up event listeners and game loop on component destruction
    return () => {
      window.removeEventListener('keydown', handleKeyDown);
      window.removeEventListener('keyup', handleKeyUp);
      clearInterval(gameLoop);
    };
  });
</script>

<style>
  .player {
    position: absolute;
    background-color: #ff0000;
  }

  .platform {
    position: absolute;
    background-color: #0000ff;
  }
</style>

<div class="game">
  <div class="player" style="left: {player.x}px; top: {player.y}px; width: {player.width}px; height: {player.height}px;"></div>

  {#each platforms as platform}
    <div class="platform" style="left: {platform.x}px; top: {platform.y}px; width: {platform.width}px; height: {platform.height}px;"></div>
  {/each}
</div>
