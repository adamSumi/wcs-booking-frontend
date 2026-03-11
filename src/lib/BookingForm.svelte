<script lang="ts">
  import { onMount } from 'svelte';

  let selectedLocation = '';
  let files: FileList;

  // UI State for submission
  let isSubmitting = false;
  let submitMessage = '';
  let isError = false;

  // Calendar State
  let availabilityMap: Record<string, string[]> = {};
  let availableDates: string[] = [];
  let isLoadingTimes = true;

  // Carousel & Selection State
  let currentDateIndex = 0;
  let timeCarouselIndex = 0;
  let selectedDate = '';
  let selectedTime = '';

  // Reactive variables that update automatically when indexes change
  $: currentDate = availableDates[currentDateIndex] || '';
  $: timesForCurrentDate = availabilityMap[currentDate] || [];
  // Slice out a maximum of 3 timeslots for the carousel
  $: visibleTimes = timesForCurrentDate.slice(timeCarouselIndex, timeCarouselIndex + 3);

  let selectedRole = '';
  let selectedContact = '';

  // Dropdown UI State
  let activeDropdown = ''; // Tracks which menu is open ('role', 'location', or 'contact')

  const toggleDropdown = (menu: string) => {
    activeDropdown = activeDropdown === menu ? '' : menu;
  };

  const selectOption = (menu: string, value: string) => {
    if (menu === 'role') selectedRole = value;
    if (menu === 'location') selectedLocation = value;
    if (menu === 'contact') selectedContact = value;
    activeDropdown = ''; // Close the menu after selection
  };

  onMount(async () => {
    try {
      const response = await fetch('http://localhost:8080/api/availability');
      if (response.ok) {
        availabilityMap = await response.json();
        // Sort keys to ensure chronological order
        availableDates = Object.keys(availabilityMap).sort();
      } else {
        console.error('Failed to fetch timeslots from backend.');
      }
    } catch (error) {
      console.error('Error fetching availability:', error);
    } finally {
      isLoadingTimes = false;
    }
  });

  // Safely parse "YYYY-MM-DD" to avoid timezone offset issues
  const formatDateDisplay = (dateStr: string) => {
    if (!dateStr) return 'Loading...';
    const [year, month, day] = dateStr.split('-');
    const date = new Date(Number(year), Number(month) - 1, Number(day));
    return date.toLocaleDateString('en-US', { weekday: 'long', month: 'short', day: 'numeric' });
  };

  // Day Navigation
  const nextDay = () => {
    if (currentDateIndex < availableDates.length - 1) {
      currentDateIndex++;
      timeCarouselIndex = 0; // Reset time carousel when day changes
    }
  };
  const prevDay = () => {
    if (currentDateIndex > 0) {
      currentDateIndex--;
      timeCarouselIndex = 0;
    }
  };

  // Time Carousel Navigation (Sliding by 1 for a smoother feel)
  const nextTimes = () => {
    if (timeCarouselIndex + 3 < timesForCurrentDate.length) {
      timeCarouselIndex += 1;
    }
  };
  const prevTimes = () => {
    if (timeCarouselIndex > 0) {
      timeCarouselIndex -= 1;
    }
  };

  // Selection Handler
  const selectTimeSlot = (time: string) => {
    selectedDate = currentDate;
    selectedTime = time;
  };

  const handleSubmit = async (e: Event) => {
    e.preventDefault();

    if (!selectedDate || !selectedTime) {
      isError = true;
      submitMessage = 'Please select a date and time for your lesson.';
      return;
    }

    isSubmitting = true;
    submitMessage = '';
    isError = false;

    const formElement = e.currentTarget as HTMLFormElement;
    const formData = new FormData(formElement);

    // Append our custom timeslot string (e.g., "2026-03-10 06:00 PM")
    formData.append('timeslot', `${selectedDate} ${selectedTime}`);

    try {
      const response = await fetch('http://localhost:8080/api/bookings', {
        method: 'POST',
        body: formData,
      });

      if (response.ok) {
        submitMessage = 'Booking successfully submitted! I will be in touch soon.';
        formElement.reset();
        selectedLocation = 'loc1';
        selectedDate = '';
        selectedTime = '';
        selectedRole = '';
        selectedContact = '';
        files = null as any;
      } else {
        const errorText = await response.text();
        throw new Error(errorText || 'Failed to submit booking.');
      }
    } catch (error) {
      console.error("Submission error:", error);
      isError = true;
      submitMessage = 'An error occurred while submitting. Please try again.';
    } finally {
      isSubmitting = false;
    }
  };
</script>

<div class="booking-container">
  <form on:submit={handleSubmit} class="booking-form">
    <div class="form-grid">

      <div class="column left-column">
        <div class="form-group">
          <span class="label-text">Select Time*</span>

          {#if isLoadingTimes}
            <div class="booking-widget-placeholder">Loading calendar...</div>
          {:else if availableDates.length === 0}
            <div class="booking-widget-placeholder">No times currently available</div>
          {:else}
            <div class="calendar-widget">

             <div class="day-picker">
                <button type="button" class="nav-btn" on:click={prevDay} disabled={currentDateIndex === 0}>&#10094;</button>
                <div class="current-date">{formatDateDisplay(currentDate)}</div>
                <button type="button" class="nav-btn" on:click={nextDay} disabled={currentDateIndex === availableDates.length - 1}>&#10095;</button>
              </div>

              <div class="time-carousel">
                <button type="button" class="nav-btn small" on:click={prevTimes} disabled={timeCarouselIndex === 0}>&#10094;</button>

                <div class="time-slots">
                  {#each visibleTimes as time}
                    <button
                      type="button"
                      class="time-slot-btn {selectedDate === currentDate && selectedTime === time ? 'selected' : ''}"
                      on:click={() => selectTimeSlot(time)}
                    >
                      {time}
                    </button>
                  {/each}
                </div>

                <button type="button" class="nav-btn small" on:click={nextTimes} disabled={timeCarouselIndex + 3 >= timesForCurrentDate.length}>&#10095;</button>
              </div>

            </div>
          {/if}
        </div>

        <div class="form-group">
          <label for="name">Full Name*</label>
          <input type="text" id="name" name="name" placeholder="John Doe" required />
        </div>

       <div class="form-group relative">
          <span class="label-text">Role</span>
          <button type="button" class="select-trigger {selectedRole !== '' ? 'has-value' : ''}" on:click={() => toggleDropdown('role')}>
            {selectedRole !== '' ? selectedRole : 'Select Role'}
          </button>

          {#if activeDropdown === 'role'}
            <div class="custom-options">
              <button type="button" class="option-btn" on:click={() => selectOption('role', 'Lead')}>Lead</button>
              <button type="button" class="option-btn" on:click={() => selectOption('role', 'Follow')}>Follow</button>
              <button type="button" class="option-btn" on:click={() => selectOption('role', 'Both')}>Both</button>
            </div>
          {/if}
          <input type="hidden" name="role" value={selectedRole} />
        </div>

        <div class="form-group relative">
          <span class="label-text">Location</span>
          <button type="button" class="select-trigger {selectedLocation !== '' ? 'has-value' : ''}" on:click={() => toggleDropdown('location')}>
            {selectedLocation !== '' ? (selectedLocation === 'loc1' ? 'Infinity Dancesport Studio' : selectedLocation === 'loc2' ? 'Atomic Ballroom' : 'Other') : 'Select Location'}
          </button>

          {#if activeDropdown === 'location'}
            <div class="custom-options">
              <button type="button" class="option-btn" on:click={() => selectOption('location', 'loc1')}>Infinity Dancesport Studio</button>
              <button type="button" class="option-btn" on:click={() => selectOption('location', 'loc2')}>Atomic Ballroom</button>
              <button type="button" class="option-btn" on:click={() => selectOption('location', 'other')}>Other</button>
            </div>
          {/if}
          <input type="hidden" name="location" value={selectedLocation} />
        </div>

        {#if selectedLocation === 'other'}
          <div class="form-group slide-down">
            <input type="text" id="other-location" name="other-location" placeholder="Please specify location..." />
          </div>
        {/if}

        <div class="contact-row">
          <div class="form-group half-width relative">
            <span class="label-text">Contact Via</span>
            <button type="button" class="select-trigger {selectedContact !== '' ? 'has-value' : ''}" on:click={() => toggleDropdown('contact')}>
              {selectedContact !== '' ? selectedContact : 'Via ...'}
            </button>

            {#if activeDropdown === 'contact'}
              <div class="custom-options">
                <button type="button" class="option-btn" on:click={() => selectOption('contact', 'Phone')}>Phone</button>
                <button type="button" class="option-btn" on:click={() => selectOption('contact', 'Instagram')}>Instagram</button>
                <button type="button" class="option-btn" on:click={() => selectOption('contact', 'Facebook')}>Facebook</button>
              </div>
            {/if}
            <input type="hidden" name="contact-via" value={selectedContact} />
          </div>
          <div class="form-group half-width">
            <label for="handle">Handle/Number</label>
            <input type="text" id="handle" name="handle" placeholder="@username or 123..." />
          </div>
        </div>

        <div class="form-group">
          <span class="label-text">Attachments</span>

          <label for="video" class="custom-file-upload">
            <svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" class="upload-icon">
              <polygon points="23 7 16 12 23 17 23 7"></polygon>
              <rect x="1" y="5" width="15" height="14" rx="2" ry="2"></rect>
            </svg>
            <span class="upload-text">
              {#if files && files.length > 0}
                {files[0].name}
              {:else}
                Click to select video (500MB max)
              {/if}
            </span>
          </label>

          <input type="file" id="video" name="video" bind:files accept="video/*" class="hidden-file-input" />
        </div>
      </div>

      <div class="column right-column">
        <div class="form-group">
          <label for="email">Email Address*</label>
          <input type="email" id="email" name="email" placeholder="email@example.com" required />
        </div>

        <div class="form-group notes-group">
          <label for="notes">Lesson Notes</label>
          <textarea id="notes" name="notes" placeholder="What would you like to focus on?"></textarea>
        </div>
      </div>

    </div>

    <div class="submit-container">
      <button type="submit" class="submit-btn" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting... (Larger videos may take longer to upload)' : 'Submit Booking'}
      </button>

      {#if submitMessage}
        <p class="status-message" class:error={isError}>
          {submitMessage}
        </p>
      {/if}
    </div>
  </form>
</div>

<style>
  .booking-container {
    width: 100%;
    max-width: 1000px;
    margin: 0 auto;
    padding-bottom: 5rem;
  }

  .form-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 4rem;
    margin-bottom: 2rem;
  }

  .column {
    display: flex;
    flex-direction: column;
    gap: 1.5rem;
  }

  .form-group {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  .contact-row {
    display: flex;
    gap: 1rem;
  }

  .half-width {
    flex: 1;
  }

  .notes-group {
    height: 100%;
  }

  label {
    font-size: 0.9rem;
    font-weight: 500;
    color: #888;
    text-transform: uppercase;
    letter-spacing: 0.05em;
  }

  input[type="text"],
  input[type="email"],
  textarea {
    width: 100%;
    background-color: transparent;
    border: none;
    border-bottom: 1px solid #333;
    color: #e7e7e7;
    padding: 0.75rem 0;
    font-family: inherit;
    font-size: 1rem;
    transition: border-color 0.2s ease, background-color 0.2s ease;
  }

  input:focus,
  textarea:focus,
  input:not(:placeholder-shown),
  textarea:not(:placeholder-shown) {
    outline: none;
    border-bottom: 1px solid #e7e7e7;
    background-color: #080808;
  }

  input::placeholder,
  textarea::placeholder {
    color: #888;
    opacity: 1;
  }

  .relative {
    position: relative;
  }

  .select-trigger {
    width: 100%;
    text-align: left;
    background-color: transparent;
    border: none;
    border-bottom: 1px solid #333;
    color: #888; /* Placeholder color */
    padding: 0.75rem 0;
    font-family: inherit;
    font-size: 1rem;
    cursor: pointer;
    transition: border-color 0.2s ease, background-color 0.2s ease, color 0.2s ease;
  }

  .select-trigger.has-value {
    color: #e7e7e7;
    border-bottom-color: #e7e7e7;
  }

  .select-trigger:focus, .select-trigger:active {
    outline: none;
    background-color: #080808;
  }

  /* The actual dropdown menu container */
  .custom-options {
    position: absolute;
    top: 100%;
    left: 0;
    width: 100%;
    background-color: #0a0a0a;
    border: 1px solid #333;
    border-top: none;
    z-index: 50; /* Keeps it layered above the fields below it */
    display: flex;
    flex-direction: column;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.5);
  }

  /* The individual items inside the menu */
  .option-btn {
    background: transparent;
    border: none;
    border-bottom: 1px solid #1a1a1a;
    color: #a7a7a7;
    padding: 0.75rem 1rem;
    text-align: left;
    cursor: pointer;
    font-family: inherit;
    font-size: 0.95rem;
    transition: all 0.2s ease;
  }

  .option-btn:last-child {
    border-bottom: none;
  }

  .option-btn:hover {
    background-color: #222;
    color: #e7e7e7;
  }

  textarea {
    resize: vertical;
    min-height: 150px;
    height: 100%;
    border: 1px solid #333;
    padding: 1rem;
  }

  textarea:focus {
    border: 1px solid #e7e7e7;
  }

  .submit-btn {
    width: 100%;
    background-color: #222;
    color: white;
    padding: 0.8rem 1.5rem; /* Reduced vertical padding from 1.25rem */
    border: 1px solid #444;
    font-size: 1.1rem;
    font-weight: 500;
    cursor: pointer;
    transition: all 0.2s ease;
    text-transform: uppercase;
    letter-spacing: 0.1em;
  }

  .submit-btn:hover {
    background-color: white;
    color: black;
  }
  .submit-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 1rem;
  }

  button:disabled {
    background-color: #333;
    color: #888;
    cursor: not-allowed;
    border-color: #333;
  }

  .status-message {
    font-size: 1rem;
    color: #a7e7a7; /* Success green matching your nav hover */
    text-align: center;
  }

  .status-message.error {
    color: #ff6b6b; /* Soft red for errors */
  }
  .calendar-widget {
    border: 1px solid #333;
    padding: 1rem;
    display: flex;
    flex-direction: column;
    gap: 0.75rem;
    background-color: #080808;
  }

  .booking-widget-placeholder {
    padding: 2rem;
    border: 1px dashed #333;
    text-align: center;
    color: #888;
  }

  .day-picker {
    display: flex;
    justify-content: space-between;
    align-items: center;
    border-bottom: 1px solid #222;
    padding-bottom: 0.75rem;
  }

  .current-date {
    font-weight: 500;
    color: #e7e7e7;
    letter-spacing: 0.05em;
  }

  .time-carousel {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 0.5rem;
  }

  .time-slots {
    display: flex;
    gap: 0.5rem;
    flex: 1;
    justify-content: center;
    overflow: hidden;
  }

  .time-slot-btn {
    background-color: transparent;
    border: 1px solid #444;
    color: #e7e7e7;
    padding: 0.5rem 1rem;
    cursor: pointer;
    transition: all 0.2s ease;
    flex: 1;
    max-width: 120px;
    font-size: 0.9rem;
  }

  .time-slot-btn:hover {
    border-color: #e7e7e7;
  }

  .time-slot-btn.selected {
    background-color: #e7e7e7;
    color: black;
    font-weight: 600;
  }

  .nav-btn {
    background: transparent; /* Forces it to stay clear */
    border: none;
    color: #a7a7a7;
    font-size: 1.5rem;
    cursor: pointer;
    transition: color 0.2s ease;
    padding: 0 0.5rem;
    outline: none; /* Removes any default browser focus rings */
  }

  .nav-btn.small {
    font-size: 1rem;
  }

  .nav-btn:hover:not(:disabled) {
    color: #e7e7e7;
  }

  .nav-btn:disabled {
    color: #333; /* Darkens the arrow to show it's inactive */
    background: transparent; /* Kills the default gray box */
    cursor: not-allowed;
  }

  label, .label-text {
    font-size: 0.9rem;
    font-weight: 500;
    color: #888;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    display: block; /* Ensures the span behaves like the label did */
  }

  .hidden-file-input {
    display: none; /* Hide the ugly native button */
  }

  .custom-file-upload {
    border: 1px dashed #444;
    background-color: #080808;
    padding: 1.5rem;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    transition: all 0.2s ease;
    color: #888;
    height: 100%;
    min-height: 120px;
  }

  .custom-file-upload:hover {
    border-color: #e7e7e7;
    color: #e7e7e7;
  }

  .upload-icon {
    margin-bottom: 0.75rem;
    color: #888;
    transition: color 0.2s ease;
  }

  .custom-file-upload:hover .upload-icon {
    color: #e7e7e7; /* Lights up white when the box is hovered */
  }

  .upload-text {
    font-size: 0.9rem;
    text-align: center;
    word-break: break-all; /* Prevents long filenames from breaking the layout */
  }

  @media (max-width: 768px) {
    .form-grid {
      grid-template-columns: 1fr;
      gap: 2rem;
    }
  }
</style>