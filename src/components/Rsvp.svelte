<script>
    import { onMount, createEventDispatcher } from 'svelte';
    export let form;

    const SHEETS_ENDPOINT = 'https://script.google.com/macros/s/AKfycbwJ2LwLcXS5SANMJFNJakHyncuXrKS7_BbEXwhrpu9VOT7U4K63wlXFXmm7Q5wijUfgZg/exec';

    const dispatch = createEventDispatcher();

    let fontLoaded = false;
    let rsvpContainer;
    let formData = {
        name: '',
        phonenumber: '',
        email: '',
        plusones: '',
        dietary: ''
    };
    let error = null;
    let success = false;
    let isSubmitting = false;

    onMount(() => {
        // Wait for Google Fonts to load
        document.fonts.ready.then(() => {
            fontLoaded = true;
        });
    });

    async function handleSubmit(event) {
        event.preventDefault();
        error = null;
        success = false;
        isSubmitting = true;

        try {
            const response = await fetch(SHEETS_ENDPOINT, {
                method: 'POST',
                mode: 'no-cors',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify(formData)
            });

            success = true;
            dispatch('success');
        } catch (err) {
            console.error('Error:', err);
            error = 'An error occurred';
        } finally {
            isSubmitting = false;
        }
    }
</script>

<div class="rsvp-container" bind:this={rsvpContainer}>
    <form on:submit={handleSubmit}>
        <div class="banner">
            <h1 class:font-loaded={fontLoaded}>FEELING THE LOVE?</h1>
        </div>
        <p class="subtitle" class:font-loaded={fontLoaded}>SEND US YOUR DEETS!</p>

        {#if form?.error}
            <p class="error-message">{form?.error}</p>
        {/if}

        {#if form?.success}
            <p class="success-message">Excited to see you! More details to follow.</p>
        {/if}

        <div class="form-field">
            <label for="name" class:font-loaded={fontLoaded}>NAME</label>
            <input id="name" name="name" type="text" required bind:value={formData.name}>
        </div>
        <div class="form-field">
            <label for="phonenumber" class:font-loaded={fontLoaded}>PHONE NUMBER</label>
            <input id="phonenumber" name="phonenumber" type="tel" required bind:value={formData.phonenumber}>
        </div>
        <div class="form-field">
            <label for="email" class:font-loaded={fontLoaded}>EMAIL</label>
            <input id="email" name="email" type="email" required bind:value={formData.email}>
        </div>
        <div class="form-field">
            <label for="plusones" class:font-loaded={fontLoaded}>PLUS ONE NAME</label>
            <input id="plusones" name="plusones" type="text" bind:value={formData.plusones}>
        </div>
        <div class="form-field">
            <label for="dietary" class:font-loaded={fontLoaded}>DIETARY REQUIREMENTS</label>
            <textarea id="dietary" name="dietary" bind:value={formData.dietary}></textarea>
        </div>
        <button type="submit" class:font-loaded={fontLoaded} class:submitting={isSubmitting} disabled={isSubmitting}>
            <span>{isSubmitting ? '' : 'RSVP'}</span>
        </button>
    </form>
</div>

<style>
    .rsvp-container {
        display: flex;
        justify-content: center;
        align-items: center;
        min-height: 100vh;
        width: 100%;
        padding: 20px;
        box-sizing: border-box;
        pointer-events: none;
    }

    .success-message {
        color: #4CAF50;
        text-align: center;
        margin-top: 5px;
        font-size: 12px;
        font-family: 'Anton', sans-serif;
    }

    form {
        width: 90%;
        max-width: 320px;
        background: linear-gradient(180deg, #3A5B8C 0%, #6B4B7C 30%, #C94B7C 60%, #E58632 100%);
        border: 2px solid #E58632;
        border-radius: 20px;
        box-shadow: 0 4px 30px rgba(229, 134, 50, 0.3);
        overflow: hidden;
        z-index: 1000;
        padding: 15px;
        display: flex;
        flex-direction: column;
        align-items: center;
        pointer-events: auto;
    }

    .banner {
        background-color: #9B2B5A;
        color: #ffffff;
        padding: 8px 15px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
        text-align: center;
        border-radius: 10px;
        width: 100%;
        box-sizing: border-box;
    }

    h1 {
        font-family: 'Rye', cursive;
        font-size: 28px;
        text-align: center;
        margin: 0;
        color: #E58632;
        opacity: 0;
        transition: opacity 0.3s ease;
    }

    h1.font-loaded {
        opacity: 1;
    }

    .subtitle {
        font-family: 'Anton', sans-serif;
        font-size: 14px;
        color: #ffffff;
        text-align: center;
        margin-bottom: 10px;
        letter-spacing: 2px;
        opacity: 0;
        transition: opacity 0.3s ease;
    }

    .subtitle.font-loaded {
        opacity: 1;
    }

    .form-field {
        width: 100%;
        margin-bottom: 10px;
        box-sizing: border-box;
    }

    label {
        font-family: 'Anton', sans-serif;
        display: block;
        margin-bottom: 2px;
        color: #ffffff;
        font-size: 12px;
        letter-spacing: 1px;
        opacity: 0;
        transition: opacity 0.3s ease;
    }

    label.font-loaded {
        opacity: 1;
    }

    input, textarea {
        width: 100%;
        padding: 8px;
        border: 2px solid #9B2B5A;
        border-radius: 8px;
        background-color: rgba(255, 255, 255, 0.95);
        font-size: 14px;
        box-sizing: border-box;
        font-family: 'Anton', sans-serif;
    }

    input:focus, textarea:focus {
        outline: none;
        border-color: #E58632;
        box-shadow: 0 0 8px rgba(229, 134, 50, 0.4);
    }

    textarea {
        height: 60px;
        resize: vertical;
    }

    button {
        width: 100%;
        height: 50px;
        margin-top: 10px;
        background-color: #E58632;
        border: 2px solid #9B2B5A;
        border-radius: 25px;
        cursor: pointer;
        position: relative;
        padding: 0;
        display: flex;
        justify-content: center;
        align-items: center;
        transition: all 0.3s ease;
        -webkit-appearance: none;
    }

    button.submitting {
        background-image: url('/assets/disco-ball.svg');
        background-size: 40px 40px;
        background-repeat: no-repeat;
        background-position: center;
        background-color: #9B2B5A;
        animation: spin 2s linear infinite;
        -webkit-animation: spin 2s linear infinite;
    }

    @keyframes spin {
        0% { transform: rotate(0deg); -webkit-transform: rotate(0deg); }
        100% { transform: rotate(360deg); -webkit-transform: rotate(360deg); }
    }

    @-webkit-keyframes spin {
        0% { -webkit-transform: rotate(0deg); }
        100% { -webkit-transform: rotate(360deg); }
    }

    button:disabled {
        cursor: not-allowed;
        opacity: 0.8;
    }

    button span {
        font-family: 'Anton', sans-serif;
        color: #1a1a1a;
        font-size: 18px;
        letter-spacing: 2px;
        transition: opacity 0.3s ease;
    }

    button.submitting span {
        opacity: 0;
    }

    button:hover:not(:disabled) {
        transform: scale(1.02);
        box-shadow: 0 4px 15px rgba(229, 134, 50, 0.4);
    }

    .error-message {
        font-family: 'Anton', sans-serif;
        color: #ff6b6b;
        text-align: center;
        margin-top: 5px;
        font-size: 12px;
    }

    .font-loaded {
        opacity: 1;
    }
</style>