<script>
    import { goto } from '$app/navigation';
    import { onMount } from 'svelte';
    import { writable } from 'svelte/store';

    let books = writable([]); // store the books in a writable store

    let pageSize = 10;
    let pageSizes = [5, 10, 20, 50];
    let currentPage = 0;
    let totalPages = 1;

    let filterTitle = '';
    let filterAuthor = '';

    onMount(() => {
        fetchBooks();
    });

    async function fetchBooks(page = currentPage, size = pageSize, title = filterTitle, author = filterAuthor) {
        const response = await fetch(`http://localhost:8000/api/books?page=${page}&size=${size}&title=${title}&author=${author}`);
        const data = await response.json();
        books.set(data.content || []);
        currentPage = data.pageable.pageNumber;
        totalPages = data.totalPages;
    }

    function navigateToEditBook(id) {
        goto(`/edit/${id}`);
    }

    function navigateToDelBook(id) {
        const confirmed = confirm('Are you sure you want to delete this book?');
        if (confirmed) {
            goto(`/delete/${id}`);
        }
    }

    function updatePageSize(event) {
        pageSize = event.target.value;
        fetchBooks(0, pageSize);
    }

    function nextPage() {
        fetchBooks(currentPage + 1);
    }

    function prevPage() {
        fetchBooks(currentPage - 1);
    }

    function applyFilters() {
        fetchBooks(0, pageSize, filterTitle, filterAuthor);
    }
</script>

<style>
    .page-controls {
        margin-top: 20px;
    }

    .page-size-select {
        margin-left: 10px;
        width: auto;
        max-width: 100px;
    }

    .filter-controls {
        margin-bottom: 20px;
    }

    .filter-controls .form-control {
        margin-right: 10px;
    }

    .filter-controls button {
        margin-left: 10px;
    }

    .table-responsive {
        margin-top: 20px;
    }

    .table th, .table td {
        vertical-align: middle;
    }

    .page-controls {
        margin-top: 20px;
        display: flex;
        justify-content: space-between;
        align-items: center;
    }

    .page-controls .btn {
        margin: 0 5px;
    }

    .page-size-select {
        margin-left: 10px;
        width: auto;
        max-width: 100px;
    }

    .d-flex.align-items-center {
        display: flex;
        align-items: center;
    }

    .d-flex.align-items-center label {
        margin-right: 10px;
    }
</style>

<h2 class="text-center my-4">Book List</h2>

<div class="filter-controls row">
    <div class="col">
        <label for="filterTitle">Title:</label>
        <input id="filterTitle" type="text" bind:value={filterTitle} class="form-control" />
    </div>
    <div class="col">
        <label for="filterAuthor">Author:</label>
        <input id="filterAuthor" type="text" bind:value={filterAuthor} class="form-control" />
    </div>
    <div class="col-auto">
        <button class="btn btn-primary mt-4" on:click={applyFilters}>Apply Filters</button>
    </div>
    <div class="col-auto">
        <button class="btn btn-success mt-4" on:click={() => goto('/add')}>Add book</button>
    </div>
</div>

<div class="table-responsive">
    <table class="table table-striped table-hover">
        <thead class="table-dark">
            <tr>
                <th scope="col">Id</th>
                <th scope="col">Title</th>
                <th scope="col">Author</th>
                <th scope="col">Edition</th>
                <th scope="col">Pages</th>
                <th scope="col">Actions</th>
            </tr>
        </thead>
        <tbody>
            {#each $books as book}
                <tr>
                    <td>{book.id}</td>
                    <td>{book.title}</td>
                    <td>{book.author}</td>
                    <td>{book.edition}</td>
                    <td>{book.numberOfPages}</td>
                    <td>
                        <button class="btn btn-primary btn-sm me-2" on:click={() => navigateToEditBook(book.id)}>Edit</button>
                        <button class="btn btn-danger btn-sm" on:click={() => navigateToDelBook(book.id)}>Delete</button>
                    </td>
                </tr>
            {/each}
        </tbody>
    </table>
</div>

<div class="page-controls">
    <button class="btn btn-secondary" on:click={prevPage} disabled={currentPage === 0}>Previous</button>
    <span>Page {currentPage + 1} of {totalPages}</span>
    <div class="d-flex align-items-center">
        <label for="pageSizeSelect">Page Size:</label>
        <select id="pageSizeSelect" class="form-select page-size-select" on:change={updatePageSize}>
            {#each pageSizes as size}
                <option value={size} selected={size === pageSize}>{size}</option>
            {/each}
        </select>
        <button class="btn btn-secondary" on:click={nextPage} disabled={currentPage >= totalPages - 1}>Next</button>
    </div>
</div>