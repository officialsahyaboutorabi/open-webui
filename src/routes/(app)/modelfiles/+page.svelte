<script lang="ts">
	import { toast } from 'svelte-sonner';
	import fileSaver from 'file-saver';
	const { saveAs } = fileSaver;

	import { onMount } from 'svelte';

	import { WEBUI_NAME, modelfiles, settings, user } from '$lib/stores';
	import { createModel, deleteModel } from '$lib/apis/ollama';
	import {
		createNewModelfile,
		deleteModelfileByTagName,
		getModelfiles
	} from '$lib/apis/modelfiles';
	import { goto } from '$app/navigation';

	let localModelfiles = [];
	let importFiles;
	let modelfilesImportInputElement: HTMLInputElement;
	const deleteModelHandler = async (tagName) => {
		let success = null;

		success = await deleteModel(localStorage.token, tagName).catch((err) => {
			toast.error(err);
			return null;
		});

		if (success) {
			toast.success(`Deleted ${tagName}`);
		}

		return success;
	};

	const deleteModelfile = async (tagName) => {
		await deleteModelHandler(tagName);
		await deleteModelfileByTagName(localStorage.token, tagName);
		await modelfiles.set(await getModelfiles(localStorage.token));
	};

	const shareModelfile = async (modelfile) => {
		toast.success('Redirecting you to OpenWebUI Community');

		const url = 'https://openwebui.com';

		const tab = await window.open(`${url}/modelfiles/create`, '_blank');
		window.addEventListener(
			'message',
			(event) => {
				if (event.origin !== url) return;
				if (event.data === 'loaded') {
					tab.postMessage(JSON.stringify(modelfile), '*');
				}
			},
			false
		);
	};

	const saveModelfiles = async (modelfiles) => {
		let blob = new Blob([JSON.stringify(modelfiles)], {
			type: 'application/json'
		});
		saveAs(blob, `modelfiles-export-${Date.now()}.json`);
	};

	onMount(() => {
		localModelfiles = JSON.parse(localStorage.getItem('modelfiles') ?? '[]');

		if (localModelfiles) {
			console.log(localModelfiles);
		}
	});
</script>

<svelte:head>
	<title>
		{`Modelfiles | ${$WEBUI_NAME}`}
	</title>
</svelte:head>

<div class="min-h-screen max-h-[100dvh] w-full flex justify-center dark:text-white">
	<div class="flex flex-col justify-between w-full overflow-y-auto">
		<div class="max-w-2xl mx-auto w-full px-3 md:px-0 my-10">
			<div class=" text-2xl font-semibold mb-3">My Modelfiles</div>

			<a class=" flex space-x-4 cursor-pointer w-full mb-2 px-3 py-2" href="/modelfiles/create">
				<div class=" self-center w-10">
					<div
						class="w-full h-10 flex justify-center rounded-full bg-transparent dark:bg-gray-700 border border-dashed border-gray-200"
					>
						<svg
							xmlns="http://www.w3.org/2000/svg"
							viewBox="0 0 24 24"
							fill="currentColor"
							class="w-6"
						>
							<path
								fill-rule="evenodd"
								d="M12 3.75a.75.75 0 01.75.75v6.75h6.75a.75.75 0 010 1.5h-6.75v6.75a.75.75 0 01-1.5 0v-6.75H4.5a.75.75 0 010-1.5h6.75V4.5a.75.75 0 01.75-.75z"
								clip-rule="evenodd"
							/>
						</svg>
					</div>
				</div>

				<div class=" self-center">
					<div class=" font-bold">Create a modelfile</div>
					<div class=" text-sm">Customize Ollama models for a specific purpose</div>
				</div>
			</a>

			<hr class=" dark:border-gray-700" />

			<div class=" my-2 mb-5">
				{#each $modelfiles as modelfile}
					<div
						class=" flex space-x-4 cursor-pointer w-full px-3 py-2 dark:hover:bg-white/5 hover:bg-black/5 rounded-xl"
					>
						<a
							class=" flex flex-1 space-x-4 cursor-pointer w-full"
							href={`/?models=${encodeURIComponent(modelfile.tagName)}`}
						>
							<div class=" self-center w-10">
								<div class=" rounded-full bg-stone-700">
									<img
										src={modelfile.imageUrl ?? '/user.png'}
										alt="modelfile profile"
										class=" rounded-full w-full h-auto object-cover"
									/>
								</div>
							</div>

							<div class=" flex-1 self-center">
								<div class=" font-bold capitalize">{modelfile.title}</div>
								<div class=" text-sm overflow-hidden text-ellipsis line-clamp-1">
									{modelfile.desc}
								</div>
							</div>
						</a>
						<div class="flex flex-row space-x-1 self-center">
							<a
								class="self-center w-fit text-sm px-2 py-2 dark:text-gray-300 dark:hover:text-white hover:bg-black/5 dark:hover:bg-white/5 rounded-xl"
								type="button"
								href={`/modelfiles/edit?tag=${encodeURIComponent(modelfile.tagName)}`}
							>
								<svg
									xmlns="http://www.w3.org/2000/svg"
									fill="none"
									viewBox="0 0 24 24"
									stroke-width="1.5"
									stroke="currentColor"
									class="w-4 h-4"
								>
									<path
										stroke-linecap="round"
										stroke-linejoin="round"
										d="m16.862 4.487 1.687-1.688a1.875 1.875 0 1 1 2.652 2.652L6.832 19.82a4.5 4.5 0 0 1-1.897 1.13l-2.685.8.8-2.685a4.5 4.5 0 0 1 1.13-1.897L16.863 4.487Zm0 0L19.5 7.125"
									/>
								</svg>
							</a>

							<button
								class="self-center w-fit text-sm px-2 py-2 dark:text-gray-300 dark:hover:text-white hover:bg-black/5 dark:hover:bg-white/5 rounded-xl"
								type="button"
								on:click={() => {
									// console.log(modelfile);
									sessionStorage.modelfile = JSON.stringify(modelfile);
									goto('/modelfiles/create');
								}}
							>
								<svg
									xmlns="http://www.w3.org/2000/svg"
									fill="none"
									viewBox="0 0 24 24"
									stroke-width="1.5"
									stroke="currentColor"
									class="w-4 h-4"
								>
									<path
										stroke-linecap="round"
										stroke-linejoin="round"
										d="M15.75 17.25v3.375c0 .621-.504 1.125-1.125 1.125h-9.75a1.125 1.125 0 0 1-1.125-1.125V7.875c0-.621.504-1.125 1.125-1.125H6.75a9.06 9.06 0 0 1 1.5.124m7.5 10.376h3.375c.621 0 1.125-.504 1.125-1.125V11.25c0-4.46-3.243-8.161-7.5-8.876a9.06 9.06 0 0 0-1.5-.124H9.375c-.621 0-1.125.504-1.125 1.125v3.5m7.5 10.375H9.375a1.125 1.125 0 0 1-1.125-1.125v-9.25m12 6.625v-1.875a3.375 3.375 0 0 0-3.375-3.375h-1.5a1.125 1.125 0 0 1-1.125-1.125v-1.5a3.375 3.375 0 0 0-3.375-3.375H9.75"
									/>
								</svg>
							</button>

							<button
								class="self-center w-fit text-sm px-2 py-2 dark:text-gray-300 dark:hover:text-white hover:bg-black/5 dark:hover:bg-white/5 rounded-xl"
								type="button"
								on:click={() => {
									shareModelfile(modelfile);
								}}
							>
								<svg
									xmlns="http://www.w3.org/2000/svg"
									fill="none"
									viewBox="0 0 24 24"
									stroke-width="1.5"
									stroke="currentColor"
									class="w-4 h-4"
								>
									<path
										stroke-linecap="round"
										stroke-linejoin="round"
										d="M7.217 10.907a2.25 2.25 0 1 0 0 2.186m0-2.186c.18.324.283.696.283 1.093s-.103.77-.283 1.093m0-2.186 9.566-5.314m-9.566 7.5 9.566 5.314m0 0a2.25 2.25 0 1 0 3.935 2.186 2.25 2.25 0 0 0-3.935-2.186Zm0-12.814a2.25 2.25 0 1 0 3.933-2.185 2.25 2.25 0 0 0-3.933 2.185Z"
									/>
								</svg>
							</button>

							<button
								class="self-center w-fit text-sm px-2 py-2 dark:text-gray-300 dark:hover:text-white hover:bg-black/5 dark:hover:bg-white/5 rounded-xl"
								type="button"
								on:click={() => {
									deleteModelfile(modelfile.tagName);
								}}
							>
								<svg
									xmlns="http://www.w3.org/2000/svg"
									fill="none"
									viewBox="0 0 24 24"
									stroke-width="1.5"
									stroke="currentColor"
									class="w-4 h-4"
								>
									<path
										stroke-linecap="round"
										stroke-linejoin="round"
										d="m14.74 9-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 0 1-2.244 2.077H8.084a2.25 2.25 0 0 1-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 0 0-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 0 1 3.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 0 0-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 0 0-7.5 0"
									/>
								</svg>
							</button>
						</div>
					</div>
				{/each}
			</div>

			<div class=" flex justify-end w-full mb-3">
				<div class="flex space-x-1">
					<input
						id="modelfiles-import-input"
						bind:this={modelfilesImportInputElement}
						bind:files={importFiles}
						type="file"
						accept=".json"
						hidden
						on:change={() => {
							console.log(importFiles);

							let reader = new FileReader();
							reader.onload = async (event) => {
								let savedModelfiles = JSON.parse(event.target.result);
								console.log(savedModelfiles);

								for (const modelfile of savedModelfiles) {
									await createNewModelfile(localStorage.token, modelfile).catch((error) => {
										return null;
									});
								}

								await modelfiles.set(await getModelfiles(localStorage.token));
							};

							reader.readAsText(importFiles[0]);
						}}
					/>

					<button
						class="flex text-xs items-center space-x-1 px-3 py-1.5 rounded-xl bg-gray-50 hover:bg-gray-100 dark:bg-gray-800 dark:hover:bg-gray-700 dark:text-gray-200 transition"
						on:click={modelfilesImportInputElement.click}
					>
						<div class=" self-center mr-2 font-medium">Import Modelfiles</div>

						<div class=" self-center">
							<svg
								xmlns="http://www.w3.org/2000/svg"
								viewBox="0 0 16 16"
								fill="currentColor"
								class="w-3.5 h-3.5"
							>
								<path
									fill-rule="evenodd"
									d="M4 2a1.5 1.5 0 0 0-1.5 1.5v9A1.5 1.5 0 0 0 4 14h8a1.5 1.5 0 0 0 1.5-1.5V6.621a1.5 1.5 0 0 0-.44-1.06L9.94 2.439A1.5 1.5 0 0 0 8.878 2H4Zm4 9.5a.75.75 0 0 1-.75-.75V8.06l-.72.72a.75.75 0 0 1-1.06-1.06l2-2a.75.75 0 0 1 1.06 0l2 2a.75.75 0 1 1-1.06 1.06l-.72-.72v2.69a.75.75 0 0 1-.75.75Z"
									clip-rule="evenodd"
								/>
							</svg>
						</div>
					</button>

					<button
						class="flex text-xs items-center space-x-1 px-3 py-1.5 rounded-xl bg-gray-50 hover:bg-gray-100 dark:bg-gray-800 dark:hover:bg-gray-700 dark:text-gray-200 transition"
						on:click={async () => {
							saveModelfiles($modelfiles);
						}}
					>
						<div class=" self-center mr-2 font-medium">Export Modelfiles</div>

						<div class=" self-center">
							<svg
								xmlns="http://www.w3.org/2000/svg"
								viewBox="0 0 16 16"
								fill="currentColor"
								class="w-3.5 h-3.5"
							>
								<path
									fill-rule="evenodd"
									d="M4 2a1.5 1.5 0 0 0-1.5 1.5v9A1.5 1.5 0 0 0 4 14h8a1.5 1.5 0 0 0 1.5-1.5V6.621a1.5 1.5 0 0 0-.44-1.06L9.94 2.439A1.5 1.5 0 0 0 8.878 2H4Zm4 3.5a.75.75 0 0 1 .75.75v2.69l.72-.72a.75.75 0 1 1 1.06 1.06l-2 2a.75.75 0 0 1-1.06 0l-2-2a.75.75 0 0 1 1.06-1.06l.72.72V6.25A.75.75 0 0 1 8 5.5Z"
									clip-rule="evenodd"
								/>
							</svg>
						</div>
					</button>
				</div>

				{#if localModelfiles.length > 0}
					<div class="flex">
						<div class=" self-center text-sm font-medium mr-4">
							{localModelfiles.length} Local Modelfiles Detected
						</div>

						<div class="flex space-x-1">
							<button
								class="self-center w-fit text-sm px-3 py-1 border dark:border-gray-600 rounded-xl flex"
								on:click={async () => {
									for (const modelfile of localModelfiles) {
										await createNewModelfile(localStorage.token, modelfile).catch((error) => {
											return null;
										});
									}

									saveModelfiles(localModelfiles);
									localStorage.removeItem('modelfiles');
									localModelfiles = JSON.parse(localStorage.getItem('modelfiles') ?? '[]');
									await modelfiles.set(await getModelfiles(localStorage.token));
								}}
							>
								<div class=" self-center mr-2 font-medium">Sync All</div>

								<div class=" self-center">
									<svg
										xmlns="http://www.w3.org/2000/svg"
										viewBox="0 0 16 16"
										fill="currentColor"
										class="w-3.5 h-3.5"
									>
										<path
											fill-rule="evenodd"
											d="M13.836 2.477a.75.75 0 0 1 .75.75v3.182a.75.75 0 0 1-.75.75h-3.182a.75.75 0 0 1 0-1.5h1.37l-.84-.841a4.5 4.5 0 0 0-7.08.932.75.75 0 0 1-1.3-.75 6 6 0 0 1 9.44-1.242l.842.84V3.227a.75.75 0 0 1 .75-.75Zm-.911 7.5A.75.75 0 0 1 13.199 11a6 6 0 0 1-9.44 1.241l-.84-.84v1.371a.75.75 0 0 1-1.5 0V9.591a.75.75 0 0 1 .75-.75H5.35a.75.75 0 0 1 0 1.5H3.98l.841.841a4.5 4.5 0 0 0 7.08-.932.75.75 0 0 1 1.025-.273Z"
											clip-rule="evenodd"
										/>
									</svg>
								</div>
							</button>

							<button
								class="self-center w-fit text-sm p-1.5 border dark:border-gray-600 rounded-xl flex"
								on:click={async () => {
									saveModelfiles(localModelfiles);

									localStorage.removeItem('modelfiles');
									localModelfiles = JSON.parse(localStorage.getItem('modelfiles') ?? '[]');
									await modelfiles.set(await getModelfiles(localStorage.token));
								}}
							>
								<div class=" self-center">
									<svg
										xmlns="http://www.w3.org/2000/svg"
										fill="none"
										viewBox="0 0 24 24"
										stroke-width="1.5"
										stroke="currentColor"
										class="w-4 h-4"
									>
										<path
											stroke-linecap="round"
											stroke-linejoin="round"
											d="M14.74 9l-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 01-2.244 2.077H8.084a2.25 2.25 0 01-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 00-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 013.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 00-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 00-7.5 0"
										/>
									</svg>
								</div>
							</button>
						</div>
					</div>
				{/if}
			</div>

			<div class=" my-16">
				<div class=" text-2xl font-semibold mb-3">Made by OpenWebUI Community</div>

				<a
					class=" flex space-x-4 cursor-pointer w-full mb-2 px-3 py-2"
					href="https://openwebui.com/"
					target="_blank"
				>
					<div class=" self-center w-10">
						<div
							class="w-full h-10 flex justify-center rounded-full bg-transparent dark:bg-gray-700 border border-dashed border-gray-200"
						>
							<svg
								xmlns="http://www.w3.org/2000/svg"
								viewBox="0 0 24 24"
								fill="currentColor"
								class="w-6"
							>
								<path
									fill-rule="evenodd"
									d="M12 3.75a.75.75 0 01.75.75v6.75h6.75a.75.75 0 010 1.5h-6.75v6.75a.75.75 0 01-1.5 0v-6.75H4.5a.75.75 0 010-1.5h6.75V4.5a.75.75 0 01.75-.75z"
									clip-rule="evenodd"
								/>
							</svg>
						</div>
					</div>

					<div class=" self-center">
						<div class=" font-bold">Discover a modelfile</div>
						<div class=" text-sm">Discover, download, and explore model presets</div>
					</div>
				</a>
			</div>
		</div>
	</div>
</div>
