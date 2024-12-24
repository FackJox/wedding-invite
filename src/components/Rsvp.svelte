<script>
    import { Turnstile } from 'svelte-turnstile';
    import { onMount, createEventDispatcher } from 'svelte';
    export let form;
    import { PUBLIC_CF_TURNSTILE_SITE_KEY, PUBLIC_DOMAIN_NAME } from '$env/static/public';

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

    onMount(() => {
        const fontZuume = new FontFace('Zuume', 'url(/zuume.woff2)');
        fontZuume.load().then(() => {
            document.fonts.add(fontZuume);
            fontLoaded = true;
        });

        const fontBern = new FontFace('Bernoru', 'url(/bernoru.woff2)');
        fontBern.load().then(() => {
            document.fonts.add(fontBern);
            fontLoaded = true;
        });
    });

    async function handleSubmit(event) {
        event.preventDefault();
        error = null;
        success = false;

        const turnstileResponse = document.querySelector('[name="cf-turnstile-response"]').value;

        try {
            const response = await fetch('/api/rsvp', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    ...formData,
                    'cf-turnstile-response': turnstileResponse
                })
            });

            const result = await response.json();

            if (response.ok) {
                success = true;
                dispatch('success');
            } else {
                error = result.error || 'An error occurred';
            }
        } catch (err) {
            console.error('Error:', err);
            error = 'An error occurred';
        }
    }
</script>

<div class="rsvp-container" bind:this={rsvpContainer}>
    <form on:submit={handleSubmit}>
        <div class="banner">
            <h1 class:font-Zuume={fontLoaded}>FEELING THE LOVE?</h1>
        </div>
        <p class:font-Bern={fontLoaded}>SEND US YOUR DEETS!</p>

        {#if form?.error}
            <p class="error-message">{form?.error}</p>
        {/if}

        {#if form?.success}
            <p class="success-message">Excited to see you! More details to follow.</p>
        {/if}

        <div class="form-field">
            <label for="name" class:font-Bern={fontLoaded}>NAME</label>
            <input id="name" name="name" type="text" required>
        </div>
        <div class="form-field">
            <label for="phonenumber" class:font-Bern={fontLoaded}>PHONE NUMBER</label>
            <input id="phonenumber" name="phonenumber" type="tel" required>
        </div>
        <div class="form-field">
            <label for="email" class:font-Bern={fontLoaded}>EMAIL</label>
            <input id="email" name="email" type="email" required>
        </div>
        <div class="form-field">
            <label for="plusones" class:font-Bern={fontLoaded}>PLUS ONE NAME</label>
            <input id="plusones" name="plusones" type="number" >
        </div>
        <div class="form-field">
            <label for="dietary" class:font-Bern={fontLoaded}>DIETARY REQUIREMENTS</label>
            <textarea id="dietary" name="dietary"></textarea>
        </div>
        <div class="form-field">

        <Turnstile siteKey={PUBLIC_CF_TURNSTILE_SITE_KEY} />
        </div>
        <button type="submit" class:font-Bern={fontLoaded}>
            <span>RSVP</span>
        </button>
    </form>
</div>

<style>
    .rsvp-container {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        width: 90%;
        max-width: 320px;
        background-color: #FEFA99;
        border: 2px solid #343233;
        border-radius: 20px;
        box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
        overflow: hidden;
        z-index: 1000;
    }

    .success-message {
        color: #4CAF50;
        text-align: center;
        margin-top: 5px;
        font-size: 12px;
    }

    form {
        padding: 15px;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .banner {
        background-color: #343233;
        color: #FEFA99;
        padding: 3px 3px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
        text-align: center;
    }

    h1 {
        font-size: 38px;
        text-align: center;
        margin: 0;
        justify-items: center;
        padding-top: 3px;
    }

    p {
        font-size: 14px;
        color: #343233;
        text-align: center;
        margin-bottom: 10px;
    }

    .form-field {
        width: 100%;
        margin-bottom: 10px;
        box-sizing: border-box;
    }

    label {
        display: block;
        margin-bottom: 2px;
        color: #343233;
        font-size: 12px;
    }

    input, textarea {
        width: 100%;
        padding: 6px;
        border: 2px solid #343233;
        border-radius: 6px;
        background-color: #FFFFFF;
        font-size: 12px;
        box-sizing: border-box;
    }

    textarea {
        height: 60px;
        resize: vertical;
    }

    /* Add these new styles for the Turnstile component */
    :global(.turnstile) {
        width: 100% !important;
        height: auto !important;
        margin-bottom: 10px;
    }

    :global(.turnstile iframe) {
        width: 100% !important;
        height: auto !important;
    }

    button {
        width: 100%;
        /* ... (keep other button styles) ... */
    }
    button {
        width: 100%;
        height: 50px;
        margin-top: 0px;
        background-color: transparent;
        background-image: url('/assets/oval.svg');
        background-size: contain;
        background-repeat: no-repeat;
        background-position: center;
        border: none;
        cursor: pointer;
        position: relative;
        padding: 0;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    button span {
        color: #FEFA99;
        font-size: 21px;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        width: 100%;
        text-align: center;
    }

    button:hover {
        opacity: 0.8;
    }
    

    .error-message {
        color: #FF0000;
        text-align: center;
        margin-top: 5px;
        font-size: 12px;
    }

    .font-Zuume {
        font-family: 'Zuume', sans-serif;
    }

    .font-Bern {
        font-family: 'Bernoru', sans-serif;
    }
</style>