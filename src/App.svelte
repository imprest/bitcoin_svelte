<script lang="ts">
  import svelteLogo from './assets/svelte.svg'
  import viteLogo from '/vite.svg'
  import Counter from './lib/Counter.svelte'

  	type UserGetPrice = { type: 'UserGetPrice' };
	type ApiReturnedPrice = {
		type: 'ApiReturnedPrice';
		data: { error: string | undefined; result: string };
	};
	type Msg = UserGetPrice | ApiReturnedPrice;

	let model = $state('Not Asked');

	function update(msg: Msg): void {
		if (msg.type === 'UserGetPrice') {
			model = 'Loading...';
			getPrice();
		} else if (msg.type === 'ApiReturnedPrice') {
			const data = msg.data;
			if (data.error) {
				model = data.error;
				return;
			}
			model = data.result;
		} else {
			return;
		}
	}

	async function getPrice() {
		try {
			const resp = await fetch('http://api.coindesk.com/v1/bpi/currentprice.json');
			if (!resp.ok)
				return update({
					type: 'ApiReturnedPrice',
					data: { error: `getPrice() had an error = resp: ${resp.status}`, result: '' }
				});
			const body = await resp.json();
			return update({
				type: 'ApiReturnedPrice',
				data: { error: undefined, result: body.bpi.USD.rate }
			});
		} catch (err) {
			return update({
				type: 'ApiReturnedPrice',
				data: { error: `getPrice() had an error : ${err}`, result: '' }
			});
		}
	}

</script>

<main>
  <div>
    <a href="https://vite.dev" target="_blank" rel="noreferrer">
      <img src={viteLogo} class="logo" alt="Vite Logo" />
    </a>
    <a href="https://svelte.dev" target="_blank" rel="noreferrer">
      <img src={svelteLogo} class="logo svelte" alt="Svelte Logo" />
    </a>
  </div>
  <h1>Vite + Svelte</h1>

  <div class="container pt-5">
    <h1>Bitcoin Price:</h1>
    <h5>It costs right now:</h5>
    <h4>{model}</h4>
    <br />
    <button onclick={() => update({ type: 'UserGetPrice' })}>Update it now</button>
  </div>


  <div class="card">
    <Counter />
  </div>

  <p>
    Check out <a href="https://github.com/sveltejs/kit#readme" target="_blank" rel="noreferrer">SvelteKit</a>, the official Svelte app framework powered by Vite!
  </p>

  <p class="read-the-docs">
    Click on the Vite and Svelte logos to learn more
  </p>
</main>

<style>
  .logo {
    height: 6em;
    padding: 1.5em;
    will-change: filter;
    transition: filter 300ms;
  }
  .logo:hover {
    filter: drop-shadow(0 0 2em #646cffaa);
  }
  .logo.svelte:hover {
    filter: drop-shadow(0 0 2em #ff3e00aa);
  }
  .read-the-docs {
    color: #888;
  }
</style>
