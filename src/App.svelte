<script lang="ts">
  // Your logic goes here
  import BookingForm from './lib/BookingForm.svelte';

  let isMenuOpen = false;

  function toggleMenu() {
    isMenuOpen = !isMenuOpen;
  }

  function closeMenu() {
    isMenuOpen = false;
  }
</script>
<svelte:head>
  <title>Adam Sumi WCS</title>
  <meta name="description" content="West Coast Swing instruction and private lessons in San Diego with Adam Sumilong." />
</svelte:head>

<header>
  <a href="#hero">
    <img src='/assets/adamsumi_logo_final.png' class="logo" alt="AdamSumi Logo" />
  </a>

  <button class="hamburger" on:click={toggleMenu} aria-label="Toggle menu">
  <span class="bar"></span>
  <span class="bar"></span>
  <span class="bar"></span>
</button>

<nav class:open={isMenuOpen}>
  <a class="nav-link" href="#about" on:click={closeMenu}>About</a>
  <a class="nav-link" href="#calendar" on:click={closeMenu}>Calendar</a>
  <a class="nav-link" href="#booking" on:click={closeMenu}>Booking</a>
</nav>

</header>

<main class="container">
  <section id="hero">

    <video class="hero-video" autoplay loop muted playsinline>
        <source src="https://storage.googleapis.com/wcs-booking-backend-assets/background_video.mp4" type="video/mp4" />
    </video>

    <div class="hero-content">
      <h1 class="section-title">Artistry in Motion</h1>
      <p class="hero-description">
        Great dances are built on clear communication and a deep understanding of the music.
        Step into a focused, collaborative learning environment that prioritizes technique, musicality,
        and the development of your personal style.
      </p>
      <a href="#booking">
        <button class="hero-btn">Book a Lesson</button>
      </a>
    </div>
  </section>

  <section id="about">
    <img src='/assets/about_bg_final.jpg' class="about-image" alt="About Section Background" />

    <div class="about-content">
      <h1 class="section-title">Behind the Motion</h1>
      <p>
        Based in San Diego, California, I am a West Coast Swing dancer, instructor, and competitor
        in the WSDC Advanced division.<br/><br/>

        Since stepping into the community three years ago, I have maintained a rigorous,
        analytical approach to my own training—focusing heavily on how physics and body
        mechanics create the aesthetic and feel of a great dance. I bring this same problem-solving
        mindset to my private lessons. I specialize in helping dancers deconstruct their movement,
        troubleshoot connection issues, and build a technically sound foundation for
        social dancing and competition.<br/><br/>

        I am available for private instruction locally in Southern California
        and regularly travel to major WCS events across the country.
      </p>
    </div>
  </section>

  <section id="calendar">
    <div class="calendar-content">
      <h1 class="section-title">Schedule and Events</h1>
      <div class="calendar-layout">
        <div class="schedule-list">
          <h3>Upcoming Travel</h3>
          <ul>
            <li>
              <strong>March 2026</strong>
                <br/>High Desert Dance Classic
            </li>
            <li>
              <strong>April 2026</strong>
                <br/>Easter Swing
                <br/>City of Angels
            </li>
            <li>
              <strong>October 2026</strong>
                <br/>Atlanta Swing Classic
            </li>
          </ul>
          <p class="local-note">
            Available for local lessons in San Diego on non-travel weekends.
          </p>
        </div>

        <div class="widget-container">
          <iframe src="https://calendar.google.com/calendar/embed?height=600&wkst=1&ctz=America%2FLos_Angeles&showPrint=0&src=OTg1MGZlNTI0OGM3N2Y1YjIyZmViNWYwNTY1NmYzMWVjNjUwM2IzZDAwMTY0NzY5OGU1OTRiMjM2NmM4MWEyZkBncm91cC5jYWxlbmRhci5nb29nbGUuY29t&color=%23e67c73"
            title="WCS Calendar"
            class="styled-calendar"
            frameborder="0"
            scrolling="no">
          </iframe>

          <a href="https://calendar.google.com/calendar/embed?height=600&wkst=1&ctz=America%2FLos_Angeles&showPrint=0&src=OTg1MGZlNTI0OGM3N2Y1YjIyZmViNWYwNTY1NmYzMWVjNjUwM2IzZDAwMTY0NzY5OGU1OTRiMjM2NmM4MWEyZkBncm91cC5jYWxlbmRhci5nb29nbGUuY29t&color=%23e67c73"
             target="_blank"
             rel="noopener noreferrer"
             class="calendar-fallback-link">
            Calendar not loading? Click here to view it directly.
          </a>
        </div>
      </div>

    </div>
  </section>

  <section id="booking">
    <div class="booking-content">
    <h1 class="section-title">Book a Lesson!</h1>
    <BookingForm />
    </div>
  </section>

</main>

<style>
  /* Your styles go here */
</style>
