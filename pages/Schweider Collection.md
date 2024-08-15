---
title: Collection History
layout: about
permalink: /Schweider_Buxton
---
## Buxton Interviews by Dorothy Schweider Collections

This hosts the collection for Dorothy Schweider's Buxton Interviews.


Sample Vignettes:

{%- assign carousel-max = include.max | default: 9 -%}
{%- assign btn-color = include.btn-color | default: "primary" -%}
{%- assign btn-text = include.btn-text | default: "View Item" -%}
{% if site.data.theme.carousel-child-objects == true %}
{%- assign carousel-items = site.data[site.metadata] | where_exp: 'item','item.objectid' | where_exp: "item","item.image_small != nil or item.image_thumb != nil"-%}
{% else %}
{%- assign carousel-items = site.data[site.metadata] | where_exp: 'item','item.objectid and item.parentid == nil' | where_exp: "item","item.image_small != nil or item.image_thumb != nil" -%}
{% endif %}

{%- comment -%}
    Set up carousel div
{%- endcomment -%}
<style>
    #imageCarousel .carousel-item { height: {{ include.height | remove: 'px' | strip | default: '300' }}px; }
</style>
<div class="card mb-3">
    {% if include.header %}<h2 class="card-header h5">{{ include.header }}</h2>{% endif %}
    <div class="card-body">
        {% if include.title %}<h2 class="card-title h5">{{ include.title }}</h2>{% endif %}
        
        <div id="imageCarousel" class="carousel slide bg-dark rounded mb-3" data-bs-ride="carousel">
            <div id="carouselIndicators" class="carousel-indicators">
                
            </div>
            <div id="carouselInner" class="carousel-inner">
                
            </div>
            <button class="carousel-control-prev" type="button" data-bs-target="#imageCarousel" data-bs-slide="prev">
                <span class="carousel-control-prev-icon" aria-hidden="true"></span>
                <span class="visually-hidden">Previous</span>
            </button>
            <button class="carousel-control-next" type="button" data-bs-target="#imageCarousel" data-bs-slide="next">
                <span class="carousel-control-next-icon" aria-hidden="true"></span>
                <span class="visually-hidden">Next</span>
            </button>
        </div>
            
    </div>
</div>
{%- comment -%}
    add slides using JS to allow for randomizing slide show
{%- endcomment -%}
<script>
    /* add item data */
    // title,objectid,image_thumb/small
    var carouselItems = [ {% for c in carousel-items %}[ {{ c.title | escape | jsonify }}, "{{ c.objectid }}", {% if c.image_small %}{{ c.image_small | relative_url | jsonify }}{% else %}{{ c.image_thumb | relative_url | jsonify }}{% endif %}, "{{ c.parentid }}", {{ c.image_alt_text | default: c.title | escape | jsonify }} ]{% unless forloop.last %}, {% endunless %}{% endfor %}];
    /* shuffle items */
    carouselItems.sort(function() { return 0.5 - Math.random() });

    /* add items to carousel */
    var carousel = document.getElementById("carouselInner");
    var carouselIndicators = document.getElementById("carouselIndicators");
    var slides = "";
    var indicators = "";
    var i, itemImg, itemLink;
    for (i=0; i < {{ carousel-items | size | at_most: carousel-max }}; i++) {
        // calculate item image location
        itemImg = carouselItems[i][2];
        // calculate item link
        if (carouselItems[i][3]){
            itemLink = '{{ '/items/' | relative_url }}' + carouselItems[i][3] + '.html#' + carouselItems[i][1];}
        else {
            itemLink = '{{ '/items/' | relative_url }}' + carouselItems[i][1] + '.html';
        }   
        // create indicator 
        indicator = `<button type="button" data-bs-target="#imageCarousel" data-bs-slide-to="${i.toString()}" ${ i == 0 ? 'class="active" aria-current="true" ' : '' }aria-label="Slide ${i.toString()}"></button>`;
        // create slide
        slide = `<div class="carousel-item py-2${ i == 0 ? ' active' : '' }">
            <img class="d-block h-100 mx-auto lazyload" alt="${carouselItems[i][4]}" data-src="${itemImg}">
            <div class="carousel-caption">
                <h3 class="carousel-item-title text-white py-2 h6">${carouselItems[i][0]}</h3>
                <a href="${itemLink}" class="btn btn-sm btn-{{ btn-color }}">{{ btn-text }}</a>
            </div></div>`;
        slides += slide;
        indicators += indicator;
    }
    // add indicators to page 
    carouselIndicators.innerHTML = indicators;
    // add slides to the page
    carousel.innerHTML = slides;
</script>

<div class="row mb-3 justify-content-center">
    <div class="col-md-8 text-center">
        <form role="search" id="lunrSearch" onsubmit="submitFilter(); return false;">
            <div class="input-group input-group-lg">
                <input type="text" class="form-control" id="filterTextBox" placeholder="Filter ... " aria-label="Search"> 
                <button class="btn btn-success" type="submit" title="Filter items" id="filterButton" >Search</button>
                <button class="btn btn-outline-secondary filter" onclick="resetFilter(); return false;" data-filter="">Reset</button>
            </div>
        </form>
        <div class="h2" id="numberOf"></div>
    </div>
    <div class="col-md-2">
        <div class="dropdown">
            <button class="btn btn-secondary mt-1 dropdown-toggle" type="button" id="browseSortButton" data-bs-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
                Sort by <span id="sortFilter">Random</span>
            </button>
            <div class="dropdown-menu browse-sort-menu" aria-labelledby="browseSortButton">
                <button class="dropdown-item browse-sort-item active" data-filter="random">Random</button>
                <button class="dropdown-item browse-sort-item" data-filter="title">Title</button>
                {% assign sort_options = site.data.config-browse | where_exp: 'i','i.sort_name != nil' %}
                {% for o in sort_options %}
                <button class="dropdown-item browse-sort-item" data-filter="{{ o.field | escape }}">{{ o.sort_name }}</button>
                {% endfor %}
            </div>
        </div>
    </div>
</div>

<div id="loadingIcon" class="text-center">
    <div class="spinner-border text-dark" role="status"><span class="visually-hidden">Loading...</span></div>
</div>

<div class="row" id="browseItems"></div>

{% if site.data.theme.browse-child-objects == true %}
{%- assign items = site.data[site.metadata] | where_exp: 'item','item.objectid' -%}
{% else %}
{%- assign items = site.data[site.metadata] | where_exp: 'item','item.objectid and item.parentid == nil' -%}
{% endif %}
{%- assign fields = site.data.config-browse -%}
<script>

/* add items */
var items = [
    {% for item in items %}
    { {% for f in fields %}{% if item[f.field] %}{{ f.field | escape | jsonify }}:{{ item[f.field] | strip | jsonify }}, {%- endif -%}{%- endfor -%}
        {% if item.image_thumb %}"img": {{ item.image_thumb | relative_url | jsonify }}, {%- endif -%} 
        {% if item.display_template %}"template": {{ item.display_template | replace: "_"," " | jsonify }}, {%- endif -%}
        {% if item.format %}"format": {{ item.format | jsonify }}, {%- endif -%}
        {% if item.image_alt_text %}"alt": {{ item.image_alt_text | escape | jsonify }}, {%- endif -%}
        "title":{{ item.title | strip | escape | jsonify }},
        {% if item.parentid %}"parent": {{ item.parentid | jsonify }}, {%- endif -%}
        "id":{{ item.objectid | jsonify }} }{% unless forloop.last %},{% endunless %}{%- endfor -%}
];

{% include js/get-icon.js %}

/* function to create cards for each item */ 
function makeCard(obj) {
    // placeholder image for lazyload
    var placeholder = "data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 3 2'%3E%3C/svg%3E";
    // find item link
    var itemHref = `{{ '/items/' | relative_url }}${ obj.parent ? obj.parent + ".html#" + obj.id : obj.id + ".html"}`; 
    // find image
    var imgSrc, thumbIcon;
    // if there is a thumb in the csv display it
    if(obj.img) {
        imgSrc = obj.img;
    // else add an icon based on display_template or format
    } else {
        thumbIcon = getIcon(obj.template,obj.format,"thumb");
    }
    var imgAlt = obj.alt ? obj.alt : obj.title;

    // start card
    var card = '<div class="item col-lg-4 col-md-6 mb-2"><div class="card">';
    // top image for items with image
    if(imgSrc) {
        card += '<a href="' + itemHref + '"> <img class="card-img-top lazyload" src="' + placeholder + '" data-src="' + imgSrc + '" alt="' + imgAlt + '"></a>';
    }
    // title
    card += '<div class="card-body text-center"> <h3 class="card-title h4"><a href="' + itemHref + '" class="text-dark">' + obj.title + '</a></h3>';
    // icon thumb for item without image
    if(thumbIcon){
        card += '<p><a href="' + itemHref + '">' + thumbIcon + '</a></p>';
    }
    // other fields
    card += '<p class="card-text">';
    {% for f in fields %}{% unless f.hidden == 'true' %}
    if(obj[{{ f.field | jsonify }}]){
    {% if f.display_name %}card += '<strong>{{ f.display_name }}:</strong> ';{% endif %}
    {% if f.btn == 'true' %}
    var btns = obj[{{ f.field | jsonify }}].split(";");
    for (var i = 0, len = btns.length; i < len; i++) {
        if(btns[i] != "") {
        card += '<a class="btn btn-sm btn-secondary m-1 text-wrap" href="{{ page.url | relative_url }}#' + encodeURIComponent(btns[i].trim()) + '">' + btns[i].trim() + '</a>';
        }
    }
    {% else %}
    card += obj[{{ f.field | jsonify }}];
    {% endif %}
    {% unless forloop.last %}card += '<br>';{% endunless %}
    }
    {% endunless %}{% endfor %}
    card += '</p>';
    // media type
    if(obj.template && obj.template != "") {
        mediaIcon = getIcon(obj.template,obj.format,"hidden");
        card += '<p class="card-text"><small><a class="btn btn-sm btn-outline-secondary" href="{{ page.url | relative_url }}#' + encodeURIComponent(obj.template) + '">' + 
        obj.template.toUpperCase() + ' ' + mediaIcon + '</a></small></p>';
    }
    // view button
    card += '<hr><a href="' + itemHref + '" class="btn btn-sm btn-light" title="link to ' + obj.title + '">View Full Record</a>';
    // close divs
    card += '</div></div></div>';
    // send back big string
    return card;
}

/* filter items function */
function filterItems(arr,q) {
    // show loading icon
    loadingIcon.classList.remove("d-none");
    // dont filter if no q 
    if (q=="") { 
        var filteredItems = arr; 
    } else {
        q = q.trim().toUpperCase(); 
        // js indexOf filter
        var filteredItems = [];
        for (var i = 0, len = arr.length; i < len; i++) {
            var val = "";
        for (var k in arr[i]) { val += arr[i][k] + " "; }
            if(val.toUpperCase().indexOf(q) != -1){
                filteredItems.push(arr[i]);
            }
        }
    }
    // add number 
    document.querySelector("#numberOf").innerHTML = filteredItems.length + " of {{ items | size }} items";
    
    // add stuff, make cards first in giant var, then add all at once to speed things up
    var cards = "";
    for (var i = 0, len = filteredItems.length; i < len; i++) {
        cards += makeCard(filteredItems[i]);
    }
    browseItemsDiv.innerHTML = cards;

    // finish
    filterButton.focus();
    loadingIcon.classList.add("d-none");
};

/* Fisher-Yates shuffle https://bost.ocks.org/mike/shuffle/ */
function shuffle(array) {
    var m = array.length, t, i;
    while (m) {
        i = Math.floor(Math.random() * m--);
        t = array[m];
        array[m] = array[i];
        array[i] = t;
    }
    return array;
}

/* init browse page */

/* randomize items once at page load */
shuffle(items);

/* set some elements */ 
var loadingIcon = document.querySelector("#loadingIcon");
var filterTextBox = document.querySelector('#filterTextBox');
var filterButton = document.querySelector("#filterButton");
var browseItemsDiv = document.querySelector("#browseItems");

/* filter if hash in initial URL */
var query = "";
if(window.location.hash) {
    query = decodeURIComponent(location.hash.substr(1));
    filterTextBox.value = query;
    filterItems(items,query);
} else {
    query = "";
    filterItems(items,query);
}

/* filter form */
function submitFilter() {
    query = filterTextBox.value;
    window.location.hash = encodeURIComponent(query);
}
/* reset filters */ 
function resetFilter() {
    query = "";
    filterTextBox.value = query;
    window.location.hash = encodeURIComponent(query);
}

/* filter if hash changes */ 
window.addEventListener("hashchange", function() {
    // read hash
    query = decodeURIComponent(location.hash.substr(1));
    filterTextBox.value = query;
    // filter
    filterItems(items,query);
});

/* item array sorting function */
function sorting(json_object, key_to_sort_by) {
    function sortByKey(a, b) {
        var x = a[key_to_sort_by];
        var y = b[key_to_sort_by];
        if (typeof x === 'string' ) { x = x.toUpperCase(); }
        if (typeof y === 'string' ) { y = y.toUpperCase(); }
        return ((x==null) ? 1: (y==null) ? -1: (x < y) ? -1 : ((x > y) ? 1 : 0));
    }
    json_object.sort(sortByKey);
};

/* add sort function on click of sort options */
var sortFilter = document.querySelector("#sortFilter");
var sortOptions = document.querySelectorAll(".browse-sort-item");
sortOptions.forEach((button) => {
  button.addEventListener("click", (event) => {
    // get the sort field
    var field = button.dataset.filter;
    var display_name = button.textContent;
    // get current filter
    var query = filterTextBox.value;
    // switch active sort option
    sortOptions.forEach((option) => { option.classList.remove("active"); } );
    button.classList.add("active");
    sortFilter.innerHTML = display_name;
    // send to sort and filter
    if (field != 'random') {
        sorting(items, field);
        filterItems(items, query);
    }
    else {
        shuffle(items);
        filterItems(items, query);
    }
  });
}); 

</script>
