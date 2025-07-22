<script lang="ts">
	export let note: { id: string; title: string; content: string; updated: number };
	export let editing: boolean;

	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	let title = note.title;
	let content = note.content;
	let localEditing = editing;

	// React when a new note is received
	$: if (note) {
		title = note.title;
		content = note.content;
	}

	function startEdit() {
		dispatch('startEdit');
		localEditing = true;
	}

	function save() {
		dispatch('save', { title: title.trim() || 'Untitled Note', content });
		localEditing = false;
	}

	function cancel() {
		title = note.title;
		content = note.content;
		dispatch('cancel');
		localEditing = false;
	}

	function remove() {
		dispatch('delete');
	}

</script>

<div class="editor-header">
	{#if !localEditing}
		<h2 class="note-title">{title}</h2>
		<button class="edit-btn" on:click={startEdit} aria-label="Edit note">Edit</button>
		<button class="del-btn" on:click={remove} aria-label="Delete note">&times;</button>
	{:else}
		<input
			class="note-title-input"
			bind:value={title}
			placeholder="Note title"
			maxlength={64}
			autocomplete="off"
		/>
		<button class="save-btn" on:click={save} aria-label="Save note">Save</button>
		<button class="cancel-btn" on:click={cancel} aria-label="Cancel edit">Cancel</button>
	{/if}
</div>
<div class="editor-body">
	{#if localEditing}
		<textarea
			class="note-content-input"
			bind:value={content}
			rows={12}
			placeholder="Start typing your note..."
			autocomplete="off"
		></textarea>
	{:else}
		<div class="note-content" style="white-space: pre-line;">
			{#if content}
				{content}
			{:else}
				<em>No content.</em>
			{/if}
		</div>
	{/if}
</div>
<div class="editor-footer">
	<small>Last edited: {new Date(note.updated).toLocaleString()}</small>
</div>

<style>
.editor-header {
	display: flex;
	align-items: center;
	gap: .7rem;
	margin-bottom: .8rem;
}
.note-title {
	font-size: 1.35rem;
	flex: 1;
	margin: 0;
	word-break: break-word;
}
.note-title-input {
	font-size: 1.35rem;
	flex: 1;
	padding: .4rem .6rem;
	border: 1.5px solid #dbeafe;
	border-radius: 5px;
}
.edit-btn, .save-btn, .cancel-btn {
	font-size: 1rem;
	padding: .35rem 1.3rem;
	border-radius: 6px;
	border: none;
	margin-left: .3rem;
	cursor: pointer;
	background: var(--primary, #3b82f6);
	color: #fff;
	transition: background .15s;
}
.save-btn {
	background: var(--accent, #f59e42);
}
.cancel-btn {
	background: #9ca3af;
	color: #fff;
}
.edit-btn:hover, .save-btn:hover, .cancel-btn:hover {
	filter: brightness(0.98) saturate(1.16);
}
.del-btn {
	color: #e11d48;
	background: transparent;
	font-size: 1.3rem;
	border: none;
	cursor: pointer;
	padding: .22rem .6rem;
	border-radius: 4px;
	margin-left: .0rem;
	transition: background .16s;
}
.del-btn:hover, .del-btn:focus {
	background: #ff2b6b;
	color: #fff;
}
.editor-body {
	margin: 0.7rem 0;
}
.note-content-input {
	font-size: 1.04rem;
	width: 100%;
	padding: .8rem .6rem;
	border: 1.5px solid #dbeafe;
	border-radius: 6px;
	min-height: 210px;
	resize: vertical;
}
.note-content {
	font-size: 1.05rem;
	color: #50546c;
}
.editor-footer {
	margin-top: .75rem;
	font-size: 0.85rem;
	color: #9ca3af;
}
@media (max-width: 700px) {
	.editor-header, .editor-footer {
		font-size: .98rem;
	}
	.note-content-input {
		min-height: 130px;
	}
}
</style>
