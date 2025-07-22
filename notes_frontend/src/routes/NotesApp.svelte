<script lang="ts">
/*
	NotesApp.svelte - Svelte UI integrated with Supabase for CRUD operations.
*/

import { onMount } from 'svelte';
import NoteList from './NoteList.svelte';
import NoteEditor from './NoteEditor.svelte';
import {
	fetchNotes as supabaseFetchNotes,
	createNote as supabaseCreateNote,
	updateNote as supabaseUpdateNote,
	deleteNote as supabaseDeleteNote,
	type Note
} from '$lib/supabaseClient';

let notes: Note[] = [];
let selectedNoteId: string | null = null;
let editing: boolean = false;
let searchTerm: string = '';

let loading: boolean = false;
let error: string | null = null;

// PUBLIC_INTERFACE
/**
 * Load notes from Supabase on mount.
 */
async function loadNotes() {
	loading = true;
	error = null;
	try {
		const data = await supabaseFetchNotes();
		notes = data.map((note) => ({
			...note,
			created: note.created || note.updated,
			updated: note.updated,
		}));
		// Auto-select first note if possible
		if (notes.length && !selectedNoteId) {
			selectedNoteId = notes[0].id;
		}
	} catch (e: unknown) {
		if (e instanceof Error) error = e.message;
		else error = 'Failed to load notes';
	} finally {
		loading = false;
	}
}

// PUBLIC_INTERFACE
/**
 * Create a new note in Supabase.
 */
async function createNote() {
	const now = new Date();
	const noteData = {
		title: 'Untitled Note',
		content: '',
		updated: now.toISOString(),
		created: now.toISOString()
	};
	loading = true;
	error = null;
	try {
		const note = await supabaseCreateNote(noteData);
		notes = [note, ...notes];
		selectedNoteId = note.id;
		editing = true;
	} catch (e: unknown) {
		if (e instanceof Error) error = e.message;
		else error = 'Failed to create note';
	} finally {
		loading = false;
	}
}

// PUBLIC_INTERFACE
/**
 * Update a note by id in Supabase.
 */
async function updateNote(id: string, updatedData: Partial<Note>) {
	loading = true;
	error = null;
	try {
		const note = await supabaseUpdateNote(id, updatedData);
		notes = notes.map((n) =>
			n.id === id ? { ...n, ...note } : n
		);
	} catch (e: unknown) {
		if (e instanceof Error) error = e.message;
		else error = 'Failed to update note';
	} finally {
		loading = false;
	}
}

// PUBLIC_INTERFACE
/**
 * Delete a note by id in Supabase.
 */
async function deleteNote(id: string) {
	if (!confirm('Delete this note?')) return;
	loading = true;
	error = null;
	try {
		await supabaseDeleteNote(id);
		const wasSelected = selectedNoteId === id;
		notes = notes.filter((n) => n.id !== id);
		if (wasSelected) {
			selectedNoteId = notes.length ? notes[0].id : null;
		}
	} catch (e: unknown) {
		if (e instanceof Error) error = e.message;
		else error = 'Failed to delete note';
	} finally {
		loading = false;
	}
}

// PUBLIC_INTERFACE
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
			n.title?.toLowerCase().includes(searchTerm.toLowerCase()) ||
			n.content?.toLowerCase().includes(searchTerm.toLowerCase())
	  )
	: notes;

// Find the selected note
$: selectedNote = notes.find((n) => n.id === selectedNoteId) ?? null;
</script>

<div class="notes-app-container">
	<aside class="sidebar">
		<div class="sidebar-header">
			<button on:click={createNote} class="create-btn" title="New note" disabled={loading}>＋</button>
			<input
				class="search"
				type="text"
				placeholder="Search notes"
				bind:value={searchTerm}
				disabled={loading}
			/>
		</div>
		{#if loading}
			<div class="loading-indicator" style="padding: 1rem; text-align:center">
				Loading...
			</div>
		{:else}
			<NoteList
				notes={filteredNotes}
				selectedId={selectedNoteId}
				on:select={e => selectNote(e.detail)}
				on:delete={e => deleteNote(e.detail)}
			/>
		{/if}
	</aside>
	<main class="main-content">
		{#if error}
			<div class="error-message">{error}</div>
		{/if}
		{#if selectedNote}
			<NoteEditor
				note={selectedNote}
				editing={editing}
				on:startEdit={startEditing}
				on:save={e => { updateNote(selectedNote.id, e.detail); stopEditing(); }}
				on:cancel={stopEditing}
				on:delete={() => deleteNote(selectedNote.id)}
			/>
		{:else if !loading}
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

.loading-indicator {
	color: #3b82f6;
	font-weight: 500;
}
.error-message {
	color: #e11d48;
	background: #fbe9f0;
	border-radius: 6px;
	padding: 0.8em 1em;
	margin-bottom: 1rem;
	text-align: center;
}
</style>
