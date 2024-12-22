<script>
    import { Turnstile } from 'svelte-turnstile';
    import { onMount } from 'svelte';
    export let form;
    import { PUBLIC_CF_TURNSTILE_SITE_KEY, PUBLIC_DOMAIN_NAME } from '$env/static/public';

    let fontLoaded = false;
    let rsvpContainer;

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
</script>

<div class="rsvp-container" bind:this={rsvpContainer}>
    <form method="POST">
        <div class="banner">
            <h1 class:font-Zuume={fontLoaded}>FEELING THE LOVE?</h1>
        </div>
        <p class:font-Bern={fontLoaded}>SEND US YOUR DEETS!</p>

        {#if form?.error}
            <p class="error-message">{form?.error}</p>
        {/if}

        {#if form?.success}
            <script>
                window.alert('Thanks for submission we will reach out shortly');
            </script>
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
            <label for="plusones" class:font-Bern={fontLoaded}>PLUS ONES?</label>
            <input id="plusones" name="plusones" type="number" required>
        </div>
        <div class="form-field">
            <label for="dietary" class:font-Bern={fontLoaded}>DIETARY REQUIREMENTS?</label>
            <textarea id="dietary" name="dietary"></textarea>
        </div>
        <Turnstile siteKey={PUBLIC_CF_TURNSTILE_SITE_KEY} />
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

    form {
        padding: 15px;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    .banner {
        background-color: #343233;
        color: #FEFA99;
        padding: 6px 3px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
    }

    h1 {
        font-size: 38px;
        text-align: center;
        margin: 0;
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

    button {
        width: 100%;
        height: 50px;
        margin-top: 10px;
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
        font-size: 28px;
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