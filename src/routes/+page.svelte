<script lang="ts">
	import { EditorView, basicSetup } from 'codemirror';
	import { javascript, javascriptLanguage } from '@codemirror/lang-javascript';
	import { onMount } from 'svelte';
	import { solarizedDark } from 'cm6-theme-solarized-dark';
	import type { CompletionContext } from '@codemirror/autocomplete';

	function myCompletions(context: CompletionContext) {
		let word = context.matchBefore(/\w*/);
		if (word?.from == word?.to && !context.explicit) return null;
		if (!word?.to) return null;
		
		return {
			from: word?.from,
			options: [
				{
					label: 'len',
					type: 'function',
					apply: 'len()',
					detail: 'function',
					info: 'returns the length of a string or an array'
				},
				{
					label: 'first',
					type: 'function',
					detail: 'function',
					info: 'returns the first element in an array',
					apply: 'first()'
				},
				{
					label: 'last',
					type: 'function',
					detail: 'function',
					info: 'returns the last element in an array',
					apply: 'last()'

				},
				{
					label: 'rest',
					type: 'function',
					detail: 'function',
					info: 'returns the elements of an array excluding the first',
					apply: 'rest()'
				},
				{
					label: 'push',
					type: 'function',
					detail: 'function',
					info: 'adds an element to the end of an array',
					apply: 'push()'
				},
				{
					label: 'pop',
					type: 'function',
					detail: 'function',
					info: 'removes the last element of an array',
					apply: 'pop()'
				},
				{
					label: 'puts',
					type: 'function',
					detail: 'function',
					info: 'prints the arguments to stdout',
					apply: 'puts()'
				}
			]
		};
	}

	let editor: EditorView;
	let output: string[] = [];
	let code: string = `let people = [
  {"name": "doe", "age": 32 },
  { "name": "jane", "age": 32 },
  {"name": "john", "age": 32}
];

let print_people = fn(people) {
  if (len(people) > 0) {
    puts(people[0]);
    print_people(rest(people));
  }
};

print_people(people);`;

	onMount(() => {
		editor = new EditorView({
			doc: code,
			extensions: [
				basicSetup,
				javascriptLanguage.data.of({ autocomplete: myCompletions }),
				javascript(),
				solarizedDark
			],
			parent: document.querySelector('#editor') as HTMLElement
		});
	});

	async function run_code(event: { currentTarget: EventTarget & HTMLFormElement }) {
		const data = new FormData(event.currentTarget);
		data.set('code', editor.state.doc.toString());
		console.log(data.get('code'));
		const response = await fetch('https://monkey-lang.onrender.com/compile', {
			method: 'POST',
			body: data
		});
		const result = await response.json();

		output = result.output;
		console.log(result);
	}
</script>

<main class="container mx-auto p-5">
	<article class="rounded shadow-md h-[75vh]" id="editor" />

	<form class="flex mb-4" on:submit|preventDefault={run_code} method="post">
		<input type="text" name="code" hidden aria-hidden id="code" bind:value={code} />

		<button type="submit" class="bg-green-400 px-7 py-2 rounded shadow-md">Run</button>
	</form>

	<article class="text-white text-left bg-gray-950 shadow-md rounded-md">
		<pre class="p-5">{output.join('\n')}</pre>
	</article>
</main>

<style>
	#editor {
		background-color: rgb(0, 43, 54);
	}
</style>
