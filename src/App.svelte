<script>
	import { onMount } from "svelte";
	let root;
	let name;
	let pt; // SVGPoint
	let svg;
	let ws;
	let me = { x: 100, y: 100, r: 20, c: "#111", selected: false };
	let others = {};

	onMount(() => {
		name = Math.floor(Math.random() * 10000);
		svg = root.querySelector("svg");
		pt = svg.createSVGPoint();
		ws = new WebSocket("wss://ornot.vote/chat/");
		ws.onopen = () => {
			ws.send(`${name},in`);
		};

		ws.onmessage = (e) => {
			const msg = e.data.split(",");
			if (msg[0] !== name) {
				if (msg.length === 3) {
					others[msg[0]] = { x: parseInt(msg[1]), y: parseInt(msg[2]) };
				}
				console.log(msg[1]);
			}
		};

		ws.onerror = (err) => {
			console.log(err);
		};

		ws.onclose = () => {
			ws.send(`${name},out`);
		};
	});

	const drag = (e) => {
		if (e.buttons === 1) {
			pt.x = e.clientX;
			pt.y = e.clientY;
			const loc = pt.matrixTransform(svg.getScreenCTM().inverse());
			me.x = loc.x;
			me.y = loc.y;
			ws.send(`${name},${me.x},${me.y}`);
		}
	};

	function circle_enter() {
		me.c = "#f00";
		const circle = root.querySelector("circle#me");
		circle.addEventListener("mousemove", drag);
	}

	function circle_leave() {
		me.c = "#111";
		const circle = root.querySelector("circle#me");
		circle.removeEventListener("mousemove", drag);
	}

	$: other_circles = Object.keys(others).map((k) => {
		return { name: k, x: others[k].x, y: others[k].y, r:10};
	});
</script>

<style>
	svg {
		background-color: #ff0;
		display:block;
		width:100%;
		height:530px;
	}
	.box {
		display: flex;
		flex-flow: column;
		height: 100%;
	}

	.box .row {
		border: 1px dotted grey;
	}

	.box .row.header {
		flex: 0 1 auto;
	}

	.box .row.content {
		flex: 1 1 auto;
	}

	.box .row.footer {
		flex: 0 1 40px;
	}
</style>

<div class="box">
	<div class="row header">
		<input placeholder="name" bind:value={name} />
		<button disabled>change name</button>
	</div>
	<div class="row content">
		<div bind:this={root}>
			<svg>
				{#each other_circles as o}
					<circle cx={o.x} cy={o.y} r={o.r} fill="#aaa" />
					<text x={o.x} y={o.y - 15} fill="#aaa" text-anchor="middle">
						{o.name}
					</text>
				{/each}
				<!-- me -->
				<circle
					cx={me.x}
					cy={me.y}
					r={me.r}
					fill={me.c}
					id="me"
					on:mouseenter={circle_enter}
					on:mouseleave={circle_leave} />
				<text x={me.x} y={me.y - 25} text-anchor="middle">{name}</text>
			</svg>
		</div>
	</div>
	<div class="row footer" />
</div>

