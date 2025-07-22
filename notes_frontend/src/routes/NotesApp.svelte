<script lang="ts">
// Note data structure
type Note = {
	id: string;
	title: string;
	content: string;
	created: number;
	updated: number;
};

import { onMount } from 'svelte';
import NoteList from './NoteList.svelte';
import NoteEditor from './NoteEditor.svelte';

let notes: Note[] = [];
let selectedNoteId: string | null = null;
let editing: boolean = false;
let searchTerm: string = '';

// PUBLIC_INTERFACE
/**
 * Load notes from localStorage.
 */
function loadNotes() {
	const stored = localStorage.getItem('notes');
	notes = stored ? JSON.parse(stored) : [];
}

/**
 * Save notes to localStorage.
 */
function saveNotes() {
	localStorage.setItem('notes', JSON.stringify(notes));
}

/**
 * Create a new note with a default title/content.
 */
function createNote() {
	const now = Date.now();
	const note: Note = {
		id: String(now),
		title: 'Untitled Note',
		content: '',
		created: now,
		updated: now
	};
	notes = [note, ...notes];
	selectedNoteId = note.id;
	editing = true;
	saveNotes();
}

/**
 * Update a note in 'notes' by id.
 */
function updateNote(id: string, updatedData: Partial<Note>) {
	notes = notes.map((n) =>
		n.id === id ? { ...n, ...updatedData, updated: Date.now() } : n
	);
	saveNotes();
}

/**
 * Delete a note by id.
 */
function deleteNote(id: string) {
	if (confirm('Delete this note?')) {
		const wasSelected = selectedNoteId === id;
		notes = notes.filter((n) => n.id !== id);
		if (wasSelected) {
			selectedNoteId = notes.length ? notes[0].id : null;
		}
		saveNotes();
	}
}

/**
 * Select a note to view or edit
 */
function selectNote(id: string) {
	selectedNoteId = id;
	editing = false;
}

function startEditing() {
	editing = true;
}

function stopEditing() {
	editing = false;
}

onMount(() => {
	loadNotes();
});

// Filter notes by search (if any)
$: filteredNotes = searchTerm
	? notes.filter(
			(n) =>
				n.title.toLowerCase().includes(searchTerm.toLowerCase()) ||
				n.content.toLowerCase().includes(searchTerm.toLowerCase())
	  )
	: notes;

// Find the selected note
$: selectedNote = notes.find((n) => n.id === selectedNoteId) ?? null;
</script>

<div class="notes-app-container">
	<aside class="sidebar">
		<div class="sidebar-header">
			<button on:click={createNote} class="create-btn" title="New note">＋</button>
			<input
				class="search"
				type="text"
				placeholder="Search notes"
				bind:value={searchTerm}
			/>
		</div>
		<NoteList
			notes={filteredNotes}
			selectedId={selectedNoteId}
			on:select={e => selectNote(e.detail)}
			on:delete={e => deleteNote(e.detail)}
		/>
	</aside>
	<main class="main-content">
		{#if selectedNote}
			<NoteEditor
				note={selectedNote}
				editing={editing}
				on:startEdit={startEditing}
				on:save={e => { updateNote(selectedNote.id, e.detail); stopEditing(); }}
				on:cancel={stopEditing}
				on:delete={() => deleteNote(selectedNote.id)}
			/>
		{:else}
			<div class="empty-state">Select or create a note to get started.</div>
		{/if}
	</main>
</div>

<style>
.notes-app-container {
	display: flex;
	height: 80vh;
	max-height: 800px;
	box-shadow: 0 2px 12px rgba(60,80,150,0.06);
	background: #fff;
	border-radius: 16px;
	overflow: hidden;
	margin: 2rem auto;
	max-width: 900px;
	min-width: 320px;
}
.sidebar {
	background-color: #f7fafc;
	border-right: 1px solid #e0e7ef;
	flex: 0 0 270px;
	display: flex;
	flex-direction: column;
}
.sidebar-header {
	display: flex;
	align-items: center;
	padding: 1rem .75rem;
	border-bottom: 1px solid #e0e7ef;
	gap: 0.5rem;
}
.create-btn {
	background-color: var(--primary, #3b82f6);
	color: #fff;
	border: none;
	font-size: 1.7rem;
	width: 2.2rem;
	height: 2.2rem;
	border-radius: 6px;
	cursor: pointer;
	transition: background .18s;
	outline: none;
	display: flex;
	align-items: center;
	justify-content: center;
}
.create-btn:hover, .create-btn:focus {
	background: var(--accent, #f59e42);
	color: #fff;
}
.search {
	flex: 1;
	padding: .55rem .8rem;
	border: 1px solid #dde3ea;
	border-radius: 9px;
	outline: none;
	transition: border .16s;
	font-size: 1rem;
	background: #fff;
}
.search:focus {
	border-color: var(--primary, #3b82f6);
}
.main-content {
	flex: 1;
	padding: 2.2rem 2rem;
	background: #fff;
	min-width: 0;
	display: flex;
	flex-direction: column;
	justify-content: flex-start;
}
.empty-state {
	color: #888;
	text-align: center;
	font-size: 1.125rem;
	margin: auto;
	opacity: 0.6;
}
@media (max-width: 700px) {
	.notes-app-container {
		flex-direction: column;
		max-height: unset;
		height: unset;
	}
	.sidebar {
		flex: none;
		width: 100%;
		border-right: none;
		border-bottom: 1px solid #e0e7ef;
	}
	.main-content {
		padding: 1.2rem 0.8rem;
	}
}
</style>
