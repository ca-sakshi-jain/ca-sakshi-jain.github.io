---
layout: default
title: "All Articles"
description: "Browse all articles on financial reporting, audit, and FP&A strategy."
permalink: /articles/
---

# All Articles

<div class="articles-container">
  <div class="search-filter-section">
    <div class="search-box">
      <input 
        type="text" 
        id="searchInput" 
        placeholder="Search by title or tags..." 
        class="search-input"
      />
      <span class="search-icon">🔍</span>
    </div>
    
    <div class="tags-filter">
      <h4>Filter by Tags:</h4>
      <div id="tagsContainer" class="tags-list">
        <!-- Tags will be dynamically inserted here -->
      </div>
      <button id="clearTagsBtn" class="btn-clear" style="display: none;">Clear Filters</button>
    </div>
  </div>

  <div class="articles-stats">
    <span id="resultsCount">Showing all articles</span>
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
      dateObj: new Date({{ post.date | date: "%Y,%m,%d" | replace: ",", "," }})
    }{{ unless forloop.last }},{{ endunless }}
    {% endfor %}
  ];

  // State management
  const state = {
    allPosts: postsData,
    filteredPosts: postsData,
    selectedTags: new Set(),
    searchQuery: '',
    currentPage: 1,
    itemsPerPage: 10
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
      <label class="tag-filter">
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
      articlesGrid.innerHTML = '<div class="no-results">No articles found. Try adjusting your filters.</div>';
      return;
    }

    articlesGrid.innerHTML = paginatedPosts.map(post => {
      const topTags = post.tags ? post.tags.slice(0, 3) : [];
      return `
        <article class="article-card">
          <div class="article-header">
            <h3 class="article-title">
              <a href="${post.url}">${post.title}</a>
            </h3>
            <div class="article-meta">
              <time class="article-date">${post.date}</time>
              <span class="article-author">by ${post.author}</span>
            </div>
          </div>
          
          <p class="article-description">${post.description}</p>
          
          <div class="article-tags">
            ${topTags.map(tag => `<span class="tag">${tag}</span>`).join('')}
            ${post.tags.length > 3 ? `<span class="tag-more">+${post.tags.length - 3}</span>` : ''}
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
      html += `<button class="page-btn" onclick="goToPage(${state.currentPage - 1})">← Previous</button>`;
    }

    // Page numbers
    for (let i = 1; i <= totalPages; i++) {
      if (i === state.currentPage) {
        html += `<span class="page-number current">${i}</span>`;
      } else if (i <= 3 || i >= totalPages - 2 || Math.abs(i - state.currentPage) <= 1) {
        html += `<button class="page-btn" onclick="goToPage(${i})">${i}</button>`;
      } else if (i === 4 || i === totalPages - 3) {
        html += `<span class="page-ellipsis">...</span>`;
      }
    }

    // Next button
    if (state.currentPage < totalPages) {
      html += `<button class="page-btn" onclick="goToPage(${state.currentPage + 1})">Next →</button>`;
    }

    paginationControls.innerHTML = html;
    window.scrollTo({ top: 0, behavior: 'smooth' });
  }

  // Update results count
  function updateResultsCount() {
    const resultsCount = document.getElementById('resultsCount');
    const total = state.filteredPosts.length;
    const totalPages = getTotalPages();

    if (state.selectedTags.size > 0 || state.searchQuery) {
      resultsCount.textContent = `Found ${total} article${total !== 1 ? 's' : ''} (Page ${state.currentPage} of ${totalPages || 1})`;
    } else {
      resultsCount.textContent = `Showing all ${total} articles (Page ${state.currentPage} of ${totalPages || 1})`;
    }
  }

  // Update clear button visibility
  function updateClearButton() {
    const clearBtn = document.getElementById('clearTagsBtn');
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
  };

  // Clear all filters
  document.getElementById('clearTagsBtn').addEventListener('click', function() {
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
  renderArticles();
  renderPagination();
  updateResultsCount();
});
</script>

<style>
.articles-container {
  max-width: 900px;
  margin: 2rem auto;
  padding: 0 1rem;
}

.search-filter-section {
  background: #f8f9fa;
  border-radius: 8px;
  padding: 1.5rem;
  margin-bottom: 2rem;
}

.search-box {
  position: relative;
  margin-bottom: 1.5rem;
}

.search-input {
  width: 100%;
  padding: 0.75rem 2.5rem 0.75rem 1rem;
  font-size: 1rem;
  border: 2px solid #ddd;
  border-radius: 6px;
  transition: border-color 0.3s;
}

.search-input:focus {
  outline: none;
  border-color: #0366d6;
}

.search-icon {
  position: absolute;
  right: 1rem;
  top: 50%;
  transform: translateY(-50%);
  color: #999;
}

.tags-filter {
  margin-top: 1.5rem;
}

.tags-filter h4 {
  margin: 0 0 0.75rem 0;
  font-size: 0.95rem;
  color: #333;
}

.tags-list {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
}

.tag-filter {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 0.75rem;
  background: white;
  border: 1px solid #ddd;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.2s;
  font-size: 0.85rem;
}

.tag-filter:hover {
  border-color: #0366d6;
  background: #f0f7ff;
}

.tag-filter input[type="checkbox"] {
  cursor: pointer;
  margin: 0;
}

.tag-filter input[type="checkbox"]:checked + .tag-label {
  font-weight: 600;
  color: #0366d6;
}

.btn-clear {
  margin-top: 1rem;
  padding: 0.5rem 1rem;
  background: #d73a49;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: background 0.2s;
}

.btn-clear:hover {
  background: #cb2431;
}

.articles-stats {
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
  border-radius: 6px;
  padding: 1.5rem;
  transition: all 0.3s;
  background: white;
}

.article-card:hover {
  border-color: #0366d6;
  box-shadow: 0 3px 12px rgba(3, 102, 214, 0.1);
}

.article-header {
  margin-bottom: 1rem;
}

.article-title {
  margin: 0 0 0.5rem 0;
  font-size: 1.3rem;
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
  gap: 1rem;
  font-size: 0.85rem;
  color: #666;
}

.article-description {
  margin: 1rem 0;
  color: #555;
  line-height: 1.6;
}

.article-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin: 1rem 0;
}

.tag {
  display: inline-block;
  padding: 0.25rem 0.6rem;
  background: #f1f3f5;
  color: #495057;
  border-radius: 12px;
  font-size: 0.8rem;
  border: 1px solid #dee2e6;
}

.tag-more {
  padding: 0.25rem 0.6rem;
  color: #666;
  font-size: 0.8rem;
  font-style: italic;
}

.read-more {
  display: inline-block;
  color: #0366d6;
  text-decoration: none;
  font-weight: 500;
  transition: all 0.2s;
}

.read-more:hover {
  color: #0256c7;
  gap: 0.25rem;
}

.no-results {
  text-align: center;
  padding: 2rem;
  color: #666;
  background: #f6f8fa;
  border-radius: 6px;
}

.pagination-section {
  text-align: center;
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
}

.page-number.current {
  background: #0366d6;
  color: white;
  border-radius: 4px;
  min-width: 2.5rem;
}

.page-ellipsis {
  color: #999;
}

@media (max-width: 600px) {
  .search-filter-section {
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
}
</style>
