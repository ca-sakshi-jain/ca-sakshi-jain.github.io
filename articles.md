---
layout: default
title: "All Articles"
description: "Browse all articles on financial reporting, audit, and FP&A strategy by CA Sakshi Jain."
permalink: /articles/
---

<div class="articles-container">
  <div class="search-filter-top">
    <div class="search-box-top">
      <input 
        type="text" 
        id="searchInput" 
        placeholder="Search articles..." 
        class="search-input-top"
      />
    </div>
    
    <div class="tags-filter-inline">
      <button class="tags-toggle" id="tagsToggle">
        <span class="toggle-label">Filter by Tags</span>
        <span class="toggle-icon">▼</span>
      </button>
      <div id="tagsContainer" class="tags-list-inline" style="display: none;">
        <!-- Tags will be dynamically inserted here -->
      </div>
      <button id="clearFiltersBtn" class="btn-clear-inline" style="display: none;">Clear All</button>
    </div>
  </div>

  <div class="articles-header">
    <span id="resultsCount">Loading articles...</span>
  </div>

  <div id="articlesGrid" class="articles-grid">
    <!-- Articles will be inserted here by JavaScript -->
  </div>

  <div class="pagination-section">
    <div id="paginationControls" class="pagination-controls">
      <!-- Pagination buttons will be inserted here -->
    </div>
  </div>
</div>

<script>
document.addEventListener('DOMContentLoaded', function() {
  // All posts data
  const postsData = [
    {% for post in site.posts %}
    {
      title: {{ post.title | jsonify }},
      url: {{ post.url | jsonify }},
      date: {{ post.date | date: "%B %d, %Y" | jsonify }},
      author: {{ post.author | jsonify }},
      description: {{ post.description | jsonify }},
      tags: {{ post.tags | jsonify }},
      dateObj: new Date({{ post.date | date: "%Y" }}, {{ post.date | date: "%m" | minus: 1 }}, {{ post.date | date: "%d" }})
    }{{ unless forloop.last }},{{ endunless }}
    {% endfor %}
  ];

  // State management
  const state = {
    allPosts: postsData.sort(function(a, b) { return b.dateObj - a.dateObj; }),
    filteredPosts: [],
    selectedTags: new Set(),
    searchQuery: '',
    currentPage: 1,
    itemsPerPage: 20
  };

  // Extract all unique tags
  function getAllTags() {
    const tagsSet = new Set();
    postsData.forEach(post => {
      if (post.tags && Array.isArray(post.tags)) {
        post.tags.forEach(tag => tagsSet.add(tag));
      }
    });
    return Array.from(tagsSet).sort();
  }

  // Render tags filter
  function renderTags() {
    const tagsContainer = document.getElementById('tagsContainer');
    const allTags = getAllTags();
    
    tagsContainer.innerHTML = allTags.map(tag => `
      <label class="tag-filter-item">
        <input 
          type="checkbox" 
          value="${tag}" 
          class="tag-checkbox"
          ${state.selectedTags.has(tag) ? 'checked' : ''}
        />
        <span class="tag-label">${tag}</span>
      </label>
    `).join('');

    // Add event listeners
    document.querySelectorAll('.tag-checkbox').forEach(checkbox => {
      checkbox.addEventListener('change', function() {
        if (this.checked) {
          state.selectedTags.add(this.value);
        } else {
          state.selectedTags.delete(this.value);
        }
        state.currentPage = 1;
        filterAndRender();
      });
    });
  }

  // Filter posts based on search and tags
  function filterPosts() {
    let filtered = state.allPosts;

    // Filter by search query
    if (state.searchQuery) {
      const query = state.searchQuery.toLowerCase();
      filtered = filtered.filter(post => {
        const matchTitle = post.title.toLowerCase().includes(query);
        const matchTags = post.tags && post.tags.some(tag => 
          tag.toLowerCase().includes(query)
        );
        const matchDesc = post.description && post.description.toLowerCase().includes(query);
        return matchTitle || matchTags || matchDesc;
      });
    }

    // Filter by selected tags
    if (state.selectedTags.size > 0) {
      filtered = filtered.filter(post => {
        if (!post.tags) return false;
        return Array.from(state.selectedTags).some(selectedTag => 
          post.tags.includes(selectedTag)
        );
      });
    }

    state.filteredPosts = filtered;
  }

  // Get paginated posts
  function getPaginatedPosts() {
    const start = (state.currentPage - 1) * state.itemsPerPage;
    const end = start + state.itemsPerPage;
    return state.filteredPosts.slice(start, end);
  }

  // Get total pages
  function getTotalPages() {
    return Math.ceil(state.filteredPosts.length / state.itemsPerPage);
  }

  // Render articles
  function renderArticles() {
    const articlesGrid = document.getElementById('articlesGrid');
    const paginatedPosts = getPaginatedPosts();

    if (paginatedPosts.length === 0) {
      articlesGrid.innerHTML = '<div class="no-results">No articles found. Try adjusting your filters or search.</div>';
      return;
    }

    articlesGrid.innerHTML = paginatedPosts.map(post => {
      const topTags = post.tags ? post.tags.slice(0, 3) : [];
      return `
        <article class="article-card">
          <h3 class="article-title">
            <a href="${post.url}">${post.title}</a>
          </h3>
          
          <div class="article-meta">
            <time class="article-date">${post.date}</time>
            <span class="article-author">by ${post.author}</span>
          </div>
          
          <p class="article-description">${post.description}</p>
          
          <div class="article-tags">
            ${topTags.map(tag => `<span class="tag">${tag}</span>`).join('')}
            ${post.tags && post.tags.length > 3 ? `<span class="tag-more">+${post.tags.length - 3} more</span>` : ''}
          </div>
          
          <a href="${post.url}" class="read-more">Read More →</a>
        </article>
      `;
    }).join('');
  }

  // Render pagination
  function renderPagination() {
    const paginationControls = document.getElementById('paginationControls');
    const totalPages = getTotalPages();

    if (totalPages <= 1) {
      paginationControls.innerHTML = '';
      return;
    }

    let html = '';

    // Previous button
    if (state.currentPage > 1) {
      html += `<button class="page-btn" onclick="window.goToPage(${state.currentPage - 1})">← Previous</button>`;
    }

    // Page numbers
    for (let i = 1; i <= totalPages; i++) {
      if (i === state.currentPage) {
        html += `<span class="page-number current">${i}</span>`;
      } else if (i <= 3 || i >= totalPages - 2 || Math.abs(i - state.currentPage) <= 1) {
        html += `<button class="page-btn" onclick="window.goToPage(${i})">${i}</button>`;
      } else if (i === 4 || i === totalPages - 3) {
        html += `<span class="page-ellipsis">...</span>`;
      }
    }

    // Next button
    if (state.currentPage < totalPages) {
      html += `<button class="page-btn" onclick="window.goToPage(${state.currentPage + 1})">Next →</button>`;
    }

    paginationControls.innerHTML = html;
  }

  // Update results count
  function updateResultsCount() {
    const resultsCount = document.getElementById('resultsCount');
    const total = state.filteredPosts.length;
    const totalPages = getTotalPages();
    const start = (state.currentPage - 1) * state.itemsPerPage + 1;
    const end = Math.min(state.currentPage * state.itemsPerPage, total);

    if (state.filteredPosts.length === 0) {
      resultsCount.textContent = 'No articles found';
    } else {
      resultsCount.textContent = `Showing ${start}-${end} of ${total} article${total !== 1 ? 's' : ''} (Page ${state.currentPage} of ${totalPages})`;
    }
  }

  // Update clear button visibility
  function updateClearButton() {
    const clearBtn = document.getElementById('clearFiltersBtn');
    if (state.selectedTags.size > 0 || state.searchQuery) {
      clearBtn.style.display = 'inline-block';
    } else {
      clearBtn.style.display = 'none';
    }
  }

  // Master filter and render function
  function filterAndRender() {
    filterPosts();
    renderArticles();
    renderPagination();
    updateResultsCount();
    updateClearButton();
  }

  // Page navigation
  window.goToPage = function(pageNum) {
    state.currentPage = pageNum;
    filterAndRender();
    window.scrollTo({ top: 0, behavior: 'smooth' });
  };

  // Toggle tags dropdown
  document.getElementById('tagsToggle').addEventListener('click', function() {
    const container = document.getElementById('tagsContainer');
    const isVisible = container.style.display !== 'none';
    container.style.display = isVisible ? 'none' : 'block';
    this.classList.toggle('active');
  });

  // Clear all filters
  document.getElementById('clearFiltersBtn').addEventListener('click', function() {
    state.selectedTags.clear();
    state.searchQuery = '';
    document.getElementById('searchInput').value = '';
    state.currentPage = 1;
    
    // Uncheck all checkboxes
    document.querySelectorAll('.tag-checkbox').forEach(cb => cb.checked = false);
    
    filterAndRender();
  });

  // Search functionality
  const searchInput = document.getElementById('searchInput');
  let searchTimeout;
  
  searchInput.addEventListener('input', function(e) {
    clearTimeout(searchTimeout);
    state.searchQuery = e.target.value;
    state.currentPage = 1;
    
    searchTimeout = setTimeout(() => {
      filterAndRender();
    }, 300);
  });

  // Initial render
  renderTags();
  filterAndRender();
});
</script>

<style>
.articles-container {
  max-width: 900px;
  margin: 0 auto;
  padding: 0 1rem;
}

.articles-container h1 {
  text-align: center;
  margin-bottom: 2rem;
}

.search-filter-top {
  background: #f8f9fa;
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 2rem;
}

.search-box-top {
  margin-bottom: 1rem;
}

.search-input-top {
  width: 100%;
  padding: 0.75rem 1rem;
  font-size: 0.95rem;
  border: 2px solid #ddd;
  border-radius: 6px;
  transition: border-color 0.3s;
}

.search-input-top:focus {
  outline: none;
  border-color: #0366d6;
}

.tags-filter-inline {
  display: flex;
  flex-wrap: wrap;
  gap: 0.75rem;
  align-items: center;
}

.tags-toggle {
  padding: 0.65rem 1rem;
  background: white;
  border: 1px solid #ddd;
  border-radius: 6px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-weight: 500;
  transition: all 0.2s;
  font-size: 0.9rem;
  white-space: nowrap;
}

.tags-toggle:hover {
  border-color: #0366d6;
  color: #0366d6;
}

.tags-toggle.active {
  border-color: #0366d6;
  background: #f0f7ff;
}

.toggle-icon {
  font-size: 0.7rem;
  transition: transform 0.3s;
}

.tags-toggle.active .toggle-icon {
  transform: rotate(180deg);
}

.tags-list-inline {
  position: absolute;
  top: 100%;
  left: 1.5rem;
  right: 1.5rem;
  width: calc(100% - 3rem);
  margin-top: 0.5rem;
  background: white;
  border: 1px solid #ddd;
  border-radius: 6px;
  padding: 1rem;
  max-height: 300px;
  overflow-y: auto;
  z-index: 10;
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

.tag-filter-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  cursor: pointer;
  font-size: 0.9rem;
  white-space: nowrap;
}

.tag-filter-item input {
  cursor: pointer;
  margin: 0;
}

.tag-filter-item:hover {
  color: #0366d6;
}

.btn-clear-inline {
  padding: 0.65rem 1rem;
  background: #d73a49;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 0.9rem;
  font-weight: 500;
  transition: background 0.2s;
  white-space: nowrap;
}

.btn-clear-inline:hover {
  background: #cb2431;
}

.articles-header {
  margin-bottom: 1.5rem;
  font-size: 0.95rem;
  color: #666;
}

.articles-grid {
  display: grid;
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.article-card {
  border: 1px solid #e1e4e8;
  border-radius: 8px;
  padding: 1.5rem;
  transition: all 0.3s;
  background: white;
}

.article-card:hover {
  border-color: #0366d6;
  box-shadow: 0 3px 12px rgba(3, 102, 214, 0.1);
}

.article-title {
  margin: 0 0 0.75rem 0;
  font-size: 1.25rem;
  line-height: 1.4;
}

.article-title a {
  color: #0366d6;
  text-decoration: none;
}

.article-title a:hover {
  text-decoration: underline;
}

.article-meta {
  display: flex;
  gap: 1.5rem;
  font-size: 0.85rem;
  color: #666;
  margin-bottom: 0.75rem;
  flex-wrap: wrap;
}

.article-description {
  margin: 1rem 0;
  color: #555;
  line-height: 1.6;
  font-size: 0.95rem;
}

.article-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin: 1rem 0;
}

.tag {
  display: inline-block;
  padding: 0.35rem 0.7rem;
  background: #f1f3f5;
  color: #495057;
  border-radius: 12px;
  font-size: 0.8rem;
  border: 1px solid #dee2e6;
}

.tag-more {
  padding: 0.35rem 0.7rem;
  color: #666;
  font-size: 0.8rem;
  font-style: italic;
}

.read-more {
  display: inline-block;
  color: #0366d6;
  text-decoration: none;
  font-weight: 500;
  margin-top: 0.5rem;
}

.read-more:hover {
  text-decoration: underline;
}

.no-results {
  text-align: center;
  padding: 3rem 2rem;
  color: #666;
  background: #f6f8fa;
  border-radius: 8px;
}

.pagination-section {
  text-align: center;
  margin-top: 2rem;
}

.pagination-controls {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.page-btn {
  padding: 0.5rem 0.75rem;
  background: white;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
  font-size: 0.9rem;
}

.page-btn:hover {
  background: #0366d6;
  color: white;
  border-color: #0366d6;
}

.page-number {
  padding: 0.5rem 0.75rem;
  font-weight: 500;
  min-width: 2.5rem;
}

.page-number.current {
  background: #0366d6;
  color: white;
  border-radius: 4px;
}

.page-ellipsis {
  color: #999;
  padding: 0 0.25rem;
}

@media (max-width: 768px) {
  .search-filter-top {
    padding: 1rem;
  }

  .article-card {
    padding: 1rem;
  }

  .article-meta {
    flex-direction: column;
    gap: 0.25rem;
  }

  .pagination-controls {
    gap: 0.25rem;
  }

  .page-btn, .page-number {
    padding: 0.4rem 0.6rem;
    font-size: 0.85rem;
  }

  .tags-list-inline {
    width: calc(100vw - 2rem);
    left: 1rem;
    right: 1rem;
  }
}
</style>

  // Extract all unique tags
  function getAllTags() {
    const tagsSet = new Set();
    postsData.forEach(post => {
      if (post.tags && Array.isArray(post.tags)) {
        post.tags.forEach(tag => tagsSet.add(tag));
      }
    });
    return Array.from(tagsSet).sort();
  }

  // Render tags filter
  function renderTags() {
    const tagsContainer = document.getElementById('tagsContainer');
    const allTags = getAllTags();
    
    tagsContainer.innerHTML = allTags.map(tag => `
      <label class="tag-filter-item">
        <input 
          type="checkbox" 
          value="${tag}" 
          class="tag-checkbox"
          ${state.selectedTags.has(tag) ? 'checked' : ''}
        />
        <span class="tag-label">${tag}</span>
      </label>
    `).join('');

    // Add event listeners
    document.querySelectorAll('.tag-checkbox').forEach(checkbox => {
      checkbox.addEventListener('change', function() {
        if (this.checked) {
          state.selectedTags.add(this.value);
        } else {
          state.selectedTags.delete(this.value);
        }
        state.currentPage = 1;
        filterAndRender();
      });
    });
  }

  // Filter posts based on search and tags
  function filterPosts() {
    let filtered = state.allPosts;

    // Filter by search query
    if (state.searchQuery) {
      const query = state.searchQuery.toLowerCase();
      filtered = filtered.filter(post => {
        const matchTitle = post.title.toLowerCase().includes(query);
        const matchTags = post.tags && post.tags.some(tag => 
          tag.toLowerCase().includes(query)
        );
        const matchDesc = post.description && post.description.toLowerCase().includes(query);
        return matchTitle || matchTags || matchDesc;
      });
    }

    // Filter by selected tags
    if (state.selectedTags.size > 0) {
      filtered = filtered.filter(post => {
        if (!post.tags) return false;
        return Array.from(state.selectedTags).some(selectedTag => 
          post.tags.includes(selectedTag)
        );
      });
    }

    state.filteredPosts = filtered;
  }

  // Get paginated posts
  function getPaginatedPosts() {
    const start = (state.currentPage - 1) * state.itemsPerPage;
    const end = start + state.itemsPerPage;
    return state.filteredPosts.slice(start, end);
  }

  // Get total pages
  function getTotalPages() {
    return Math.ceil(state.filteredPosts.length / state.itemsPerPage);
  }

  // Render articles
  function renderArticles() {
    const articlesGrid = document.getElementById('articlesGrid');
    const paginatedPosts = getPaginatedPosts();

    if (paginatedPosts.length === 0) {
      articlesGrid.innerHTML = '<div class="no-results">No articles found. Try adjusting your filters or search.</div>';
      return;
    }

    articlesGrid.innerHTML = paginatedPosts.map(post => {
      const topTags = post.tags ? post.tags.slice(0, 3) : [];
      return `
        <article class="article-card">
          <h3 class="article-title">
            <a href="${post.url}">${post.title}</a>
          </h3>
          
          <div class="article-meta">
            <time class="article-date">${post.date}</time>
            <span class="article-author">by ${post.author}</span>
          </div>
          
          <p class="article-description">${post.description}</p>
          
          <div class="article-tags">
            ${topTags.map(tag => `<span class="tag">${tag}</span>`).join('')}
            ${post.tags && post.tags.length > 3 ? `<span class="tag-more">+${post.tags.length - 3} more</span>` : ''}
          </div>
          
          <a href="${post.url}" class="read-more">Read More →</a>
        </article>
      `;
    }).join('');
  }

  // Render pagination
  function renderPagination() {
    const paginationControls = document.getElementById('paginationControls');
    const totalPages = getTotalPages();

    if (totalPages <= 1) {
      paginationControls.innerHTML = '';
      return;
    }

    let html = '';

    // Previous button
    if (state.currentPage > 1) {
      html += `<button class="page-btn" onclick="window.goToPage(${state.currentPage - 1})">← Previous</button>`;
    }

    // Page numbers
    for (let i = 1; i <= totalPages; i++) {
      if (i === state.currentPage) {
        html += `<span class="page-number current">${i}</span>`;
      } else if (i <= 3 || i >= totalPages - 2 || Math.abs(i - state.currentPage) <= 1) {
        html += `<button class="page-btn" onclick="window.goToPage(${i})">${i}</button>`;
      } else if (i === 4 || i === totalPages - 3) {
        html += `<span class="page-ellipsis">...</span>`;
      }
    }

    // Next button
    if (state.currentPage < totalPages) {
      html += `<button class="page-btn" onclick="window.goToPage(${state.currentPage + 1})">Next →</button>`;
    }

    paginationControls.innerHTML = html;
  }

  // Update results count
  function updateResultsCount() {
    const resultsCount = document.getElementById('resultsCount');
    const total = state.filteredPosts.length;
    const totalPages = getTotalPages();
    const start = (state.currentPage - 1) * state.itemsPerPage + 1;
    const end = Math.min(state.currentPage * state.itemsPerPage, total);

    if (state.filteredPosts.length === 0) {
      resultsCount.textContent = 'No articles found';
    } else {
      resultsCount.textContent = `Showing ${start}-${end} of ${total} article${total !== 1 ? 's' : ''} (Page ${state.currentPage} of ${totalPages})`;
    }
  }

  // Update clear button visibility
  function updateClearButton() {
    const clearBtn = document.getElementById('clearFiltersBtn');
    if (state.selectedTags.size > 0 || state.searchQuery) {
      clearBtn.style.display = 'inline-block';
    } else {
      clearBtn.style.display = 'none';
    }
  }

  // Master filter and render function
  function filterAndRender() {
    filterPosts();
    renderArticles();
    renderPagination();
    updateResultsCount();
    updateClearButton();
  }

  // Page navigation
  window.goToPage = function(pageNum) {
    state.currentPage = pageNum;
    filterAndRender();
    window.scrollTo({ top: 0, behavior: 'smooth' });
  };

  // Toggle tags dropdown
  document.getElementById('tagsToggle').addEventListener('click', function() {
    const container = document.getElementById('tagsContainer');
    const isVisible = container.style.display !== 'none';
    container.style.display = isVisible ? 'none' : 'block';
    this.classList.toggle('active');
  });

  // Clear all filters
  document.getElementById('clearFiltersBtn').addEventListener('click', function() {
    state.selectedTags.clear();
    state.searchQuery = '';
    document.getElementById('searchInput').value = '';
    state.currentPage = 1;
    
    // Uncheck all checkboxes
    document.querySelectorAll('.tag-checkbox').forEach(cb => cb.checked = false);
    
    filterAndRender();
  });

  // Search functionality
  const searchInput = document.getElementById('searchInput');
  let searchTimeout;
  
  searchInput.addEventListener('input', function(e) {
    clearTimeout(searchTimeout);
    state.searchQuery = e.target.value;
    state.currentPage = 1;
    
    searchTimeout = setTimeout(() => {
      filterAndRender();
    }, 300);
  });

  // Initial render
  renderTags();
  filterAndRender();
});
</script>

<style>
.articles-wrapper {
  display: flex;
  gap: 2rem;
  max-width: 1200px;
  margin: 2rem auto;
  padding: 0 1rem;
}

.articles-sidebar {
  flex: 0 0 280px;
  background: #f8f9fa;
  border-radius: 8px;
  padding: 1.5rem;
  height: fit-content;
  position: sticky;
  top: 2rem;
}

.search-box-sidebar {
  margin-bottom: 1.5rem;
}

.search-input-sidebar {
  width: 100%;
  padding: 0.75rem;
  font-size: 0.95rem;
  border: 2px solid #ddd;
  border-radius: 6px;
  transition: border-color 0.3s;
}

.search-input-sidebar:focus {
  outline: none;
  border-color: #0366d6;
}

.tags-filter-section {
  margin-bottom: 1rem;
}

.tags-toggle {
  width: 100%;
  padding: 0.75rem;
  background: white;
  border: 1px solid #ddd;
  border-radius: 6px;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-weight: 500;
  transition: all 0.2s;
}

.tags-toggle:hover {
  border-color: #0366d6;
  color: #0366d6;
}

.tags-toggle.active {
  border-color: #0366d6;
  background: #f0f7ff;
}

.toggle-icon {
  font-size: 0.75rem;
  transition: transform 0.3s;
}

.tags-toggle.active .toggle-icon {
  transform: rotate(180deg);
}

.tags-list-sidebar {
  margin-top: 0.75rem;
  padding: 0.75rem 0;
  max-height: 400px;
  overflow-y: auto;
  border-top: 1px solid #eee;
}

.tag-filter-item {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 0;
  cursor: pointer;
  font-size: 0.9rem;
}

.tag-filter-item input {
  cursor: pointer;
  margin: 0;
}

.tag-filter-item:hover {
  color: #0366d6;
}

.btn-clear-sidebar {
  width: 100%;
  padding: 0.65rem;
  background: #d73a49;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.9rem;
  font-weight: 500;
  transition: background 0.2s;
  margin-top: 1rem;
}

.btn-clear-sidebar:hover {
  background: #cb2431;
}

.articles-main {
  flex: 1;
  min-width: 0;
}

.articles-header {
  margin-bottom: 1.5rem;
  font-size: 0.95rem;
  color: #666;
}

.articles-grid {
  display: grid;
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.article-card {
  border: 1px solid #e1e4e8;
  border-radius: 8px;
  padding: 1.5rem;
  transition: all 0.3s;
  background: white;
}

.article-card:hover {
  border-color: #0366d6;
  box-shadow: 0 3px 12px rgba(3, 102, 214, 0.1);
}

.article-title {
  margin: 0 0 0.75rem 0;
  font-size: 1.25rem;
  line-height: 1.4;
}

.article-title a {
  color: #0366d6;
  text-decoration: none;
}

.article-title a:hover {
  text-decoration: underline;
}

.article-meta {
  display: flex;
  gap: 1.5rem;
  font-size: 0.85rem;
  color: #666;
  margin-bottom: 0.75rem;
  flex-wrap: wrap;
}

.article-description {
  margin: 1rem 0;
  color: #555;
  line-height: 1.6;
  font-size: 0.95rem;
}

.article-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin: 1rem 0;
}

.tag {
  display: inline-block;
  padding: 0.35rem 0.7rem;
  background: #f1f3f5;
  color: #495057;
  border-radius: 12px;
  font-size: 0.8rem;
  border: 1px solid #dee2e6;
}

.tag-more {
  padding: 0.35rem 0.7rem;
  color: #666;
  font-size: 0.8rem;
  font-style: italic;
}

.read-more {
  display: inline-block;
  color: #0366d6;
  text-decoration: none;
  font-weight: 500;
  margin-top: 0.5rem;
}

.read-more:hover {
  text-decoration: underline;
}

.no-results {
  text-align: center;
  padding: 3rem 2rem;
  color: #666;
  background: #f6f8fa;
  border-radius: 8px;
}

.pagination-section {
  text-align: center;
  margin-top: 2rem;
}

.pagination-controls {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.page-btn {
  padding: 0.5rem 0.75rem;
  background: white;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.2s;
  font-size: 0.9rem;
}

.page-btn:hover {
  background: #0366d6;
  color: white;
  border-color: #0366d6;
}

.page-number {
  padding: 0.5rem 0.75rem;
  font-weight: 500;
  min-width: 2.5rem;
}

.page-number.current {
  background: #0366d6;
  color: white;
  border-radius: 4px;
}

.page-ellipsis {
  color: #999;
  padding: 0 0.25rem;
}

@media (max-width: 768px) {
  .articles-wrapper {
    flex-direction: column;
    gap: 1.5rem;
  }

  .articles-sidebar {
    flex: none;
    position: static;
    width: 100%;
  }

  .article-meta {
    flex-direction: column;
    gap: 0.25rem;
  }

  .pagination-controls {
    gap: 0.25rem;
  }

  .page-btn, .page-number {
    padding: 0.4rem 0.6rem;
    font-size: 0.85rem;
  }
}
</style>

