<script lang="ts">
	export let notes: Array<{ id: string; title: string; content: string; updated: number }>;
	export let selectedId: string | null;
	import { createEventDispatcher } from 'svelte';

	const dispatch = createEventDispatcher();

	function selectNote(id: string) {
		dispatch('select', id);
	}
	function deleteNote(id: string, e: Event) {
		e.stopPropagation();
		dispatch('delete', id);
	}
</script>

<ul class="note-list">
	{#if notes.length === 0}
		<li class="no-notes">No notes yet</li>
	{:else}
		{#each notes as note (note.id)}
			<li class="note-list-item {selectedId === note.id ? 'active' : ''}">
				<button
					class="note-select-btn"
					aria-current={selectedId === note.id ? 'true' : undefined}
					on:click={() => selectNote(note.id)}
				>
					<span class="title">{note.title || '(Untitled)'}</span>
					<small class="sub">{new Date(note.updated).toLocaleString()}</small>
				</button>
				<button class="del-btn" on:click={(e)=>deleteNote(note.id,e)} aria-label="Delete note">&times;</button>
			</li>
		{/each}
	{/if}
</ul>

<style>
.note-list {
	list-style: none;
	margin: 0;
	padding: 0;
	width: 100%;
	overflow-y: auto;
}
.no-notes {
	color: #aaa;
	padding: 1.4rem .8rem;
	text-align: center;
	opacity: 0.5;
}
.note-list-item {
	padding: 0.95rem 0.7rem 0.95rem 1.05rem;
	border: none;
	border-bottom: 1px solid #e5edf7;
	display: flex;
	align-items: center;
	justify-content: space-between;
	cursor: pointer;
	transition: background .14s;
	background: none;
}
.note-list-item:last-child {
	border-bottom: 0;
}
.note-list-item.active {
	background: var(--primary, #3b82f6);
	color: #fff;
}
.note-list-item .title {
	font-weight: 500;
	font-size: 1rem;
	max-width: 150px;
	white-space: nowrap;
	text-overflow: ellipsis;
	overflow: hidden;
	display: block;
}
.note-list-item .sub {
	font-size: 0.78rem;
	color: #7e889b;
	margin-left: 2px;
}
.note-list-item.active .sub {
	color: #e9ecf7;
}
.note-select-btn {
	display: flex;
	align-items: center;
	justify-content: flex-start;
	gap: 0.6rem;
	flex: 1;
	background: none;
	border: none;
	color: inherit;
	font: inherit;
	width: 100%;
	padding: 0;
	cursor: pointer;
	outline: none;
	text-align: left;
	transition: background .14s;
	border-radius: 4px;
}
.note-select-btn[aria-current="true"], .note-list-item.active .note-select-btn {
	background: var(--primary, #3b82f6);
	color: #fff;
}
.note-select-btn:hover, .note-select-btn:focus {
	background: #e0e7ef;
}
.del-btn {
	border: none;
	background: transparent;
	font-size: 1.3rem;
	color: #cbd5e1;
	cursor: pointer;
	margin-left: .5rem;
	transition: color .16s;
	padding: 1px 4px 2px 4px;
	border-radius: 4px;
}
.del-btn:hover,.del-btn:focus {
	background: #e11d48;
	color: #fff;
}
.note-list-item.active .del-btn {
	color: #fff;
}
@media (max-width: 700px) {
	.note-list-item .title {
		max-width: 90px;
		font-size: 0.97rem;
	}
}
</style>
