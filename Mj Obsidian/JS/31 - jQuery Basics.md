# jQuery Basics

## Overview
jQuery is a fast, small, and feature-rich JavaScript library that simplifies HTML document traversal, manipulation, event handling, animation, and Ajax interactions. Although modern JavaScript and frameworks have reduced jQuery's necessity, it remains widely used and understanding it is valuable for maintaining legacy code and rapid prototyping.

## Installation and Setup

### CDN Installation
```html
<!-- jQuery 3.x (latest) -->
<script src="https://code.jquery.com/jquery-3.7.1.min.js" 
        integrity="sha256-/JqT3SQfawRcv/BIHPThkBvs0OEvtFFmqPF/lYI/Cxo=" 
        crossorigin="anonymous"></script>

<!-- jQuery 2.x (IE8 support) -->
<script src="https://code.jquery.com/jquery-2.2.4.min.js"></script>

<!-- jQuery 1.x (IE6-8 support) -->
<script src="https://code.jquery.com/jquery-1.12.4.min.js"></script>

<!-- jQuery UI (optional - for enhanced widgets) -->
<script src="https://code.jquery.com/ui/1.13.2/jquery-ui.min.js"></script>
<link rel="stylesheet" href="https://code.jquery.com/ui/1.13.2/themes/ui-lightness/jquery-ui.css">
```

### Local Installation
```bash
# NPM installation
npm install jquery

# Bower installation (legacy)
bower install jquery

# Download and include
# Download from https://jquery.com/download/
```

### Including jQuery
```html
<!DOCTYPE html>
<html>
<head>
    <title>jQuery Example</title>
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
</head>
<body>
    <h1>Hello jQuery!</h1>
    
    <script>
        // jQuery code here
        $(document).ready(function() {
            console.log("jQuery is ready!");
        });
    </script>
</body>
</html>
```

## jQuery Basics

### The jQuery Function ($)
```javascript
// jQuery function aliases
jQuery === $; // true

// Different ways to use $()
$('selector');           // Select elements
$(element);              // Wrap DOM element
$(function);             // Document ready
$('<p>Hello</p>');      // Create element
$(document);             // Wrap document

// No conflict mode (if $ is used by another library)
var jq = jQuery.noConflict();
jq(document).ready(function() {
    jq('h1').hide();
});

// Document ready - equivalent to DOMContentLoaded
$(document).ready(function() {
    console.log("DOM is ready!");
    // jQuery code here
});

// Shorthand for document ready
$(function() {
    console.log("DOM is ready! (shorthand)");
});

// Window load - equivalent to window.onload
$(window).on('load', function() {
    console.log("All resources loaded!");
});

// Check if jQuery is loaded
if (typeof jQuery !== 'undefined') {
    console.log("jQuery version:", jQuery.fn.jquery);
} else {
    console.log("jQuery not loaded");
}
```

### Basic Selectors
```javascript
// Element selectors
$('p');                  // All paragraphs
$('div');                // All divs
$('h1, h2, h3');        // Multiple elements

// Class selectors
$('.className');         // Elements with class
$('p.highlight');        // Paragraphs with highlight class
$('.class1.class2');     // Elements with both classes

// ID selectors
$('#myId');              // Element with specific ID
$('div#container');      // Div with specific ID

// Attribute selectors
$('[data-id]');          // Elements with data-id attribute
$('[type="text"]');      // Input elements with type="text"
$('[href^="https"]');    // Links starting with https
$('[src$=".jpg"]');      // Images ending with .jpg
$('[title*="jQuery"]');  // Elements containing "jQuery" in title

// Pseudo-selectors
$(':first');             // First element
$(':last');              // Last element
$(':even');              // Even-indexed elements (0, 2, 4...)
$(':odd');               // Odd-indexed elements (1, 3, 5...)
$(':eq(2)');             // Element at index 2
$(':gt(2)');             // Elements with index > 2
$(':lt(2)');             // Elements with index < 2

// Form selectors
$(':input');             // All input elements
$(':text');              // Text inputs
$(':password');          // Password inputs
$(':radio');             // Radio buttons
$(':checkbox');          // Checkboxes
$(':submit');            // Submit buttons
$(':checked');           // Checked radio/checkbox
$(':selected');          // Selected options
$(':disabled');          // Disabled elements
$(':enabled');           // Enabled elements

// Visibility selectors
$(':visible');           // Visible elements
$(':hidden');            // Hidden elements

// Content selectors
$(':contains("text")');  // Elements containing text
$(':empty');             // Empty elements
$(':has("p")');          // Elements containing paragraphs

// Hierarchy selectors
$('div p');              // Descendant selector (all p in div)
$('div > p');            // Child selector (direct p children of div)
$('h1 + p');             // Adjacent sibling (p immediately after h1)
$('h1 ~ p');             // General sibling (all p siblings after h1)

// Examples with context
$(document).ready(function() {
    // Select all paragraphs with class 'intro'
    $('.intro').css('color', 'blue');
    
    // Select the first list item
    $('li:first').addClass('first-item');
    
    // Select all external links
    $('a[href^="http"]').attr('target', '_blank');
    
    // Select all required form fields
    $('input[required]').addClass('required-field');
});
```

## DOM Manipulation

### Getting and Setting Content
```javascript
// Text content
$('#myDiv').text();                    // Get text
$('#myDiv').text('New text');          // Set text
$('p').text('Same text for all');     // Set text for all paragraphs

// HTML content
$('#myDiv').html();                    // Get HTML
$('#myDiv').html('<strong>Bold</strong>'); // Set HTML
$('div').html('<p>New paragraph</p>'); // Set HTML for all divs

// Form values
$('#myInput').val();                   // Get input value
$('#myInput').val('New value');        // Set input value
$('input[type="text"]').val('');      // Clear all text inputs

// Select box values
$('#mySelect').val();                  // Get selected value
$('#mySelect').val('option2');         // Set selected value
$('#mySelect').val(['opt1', 'opt2']);  // Set multiple selections

// Checkbox and radio values
$('#myCheckbox').val();                // Get value attribute
$('#myCheckbox').prop('checked');      // Get checked state
$('#myCheckbox').prop('checked', true); // Set checked state

// Examples
$(document).ready(function() {
    // Display current values
    console.log('Page title:', $('title').text());
    console.log('First paragraph:', $('p:first').text());
    
    // Update content
    $('#welcome').text('Welcome to our site!');
    $('#content').html('<p>This is <em>dynamic</em> content.</p>');
    
    // Form manipulation
    $('#userEmail').val('user@example.com');
    $('#newsletter').prop('checked', true);
    
    // Batch operations
    $('.price').text('$9.99');
    $('input[type="text"]').val('');
});
```

### Attributes and Properties
```javascript
// Attributes (attr)
$('#myImg').attr('src');               // Get src attribute
$('#myImg').attr('src', 'new.jpg');    // Set src attribute
$('#myLink').attr('href', 'https://example.com'); // Set href

// Multiple attributes
$('#myImg').attr({
    'src': 'image.jpg',
    'alt': 'Description',
    'class': 'responsive'
});

// Remove attributes
$('#myImg').removeAttr('title');       // Remove title attribute

// Properties (prop) - for boolean properties
$('#myCheckbox').prop('checked');      // Get checked property
$('#myCheckbox').prop('checked', true); // Set checked property
$('#myInput').prop('disabled', false); // Enable input

// Data attributes
$('#myDiv').data('userId');            // Get data-user-id
$('#myDiv').data('userId', 123);       // Set data-user-id
$('#myDiv').data('user-name', 'John'); // Set data-user-name

// Multiple data attributes
$('#myDiv').data({
    'userId': 123,
    'userName': 'John',
    'userRole': 'admin'
});

// Remove data
$('#myDiv').removeData('userId');      // Remove data attribute

// Examples
$(document).ready(function() {
    // Image gallery
    $('.thumbnail').attr({
        'data-large': function() {
            return $(this).attr('src').replace('thumb_', 'large_');
        },
        'alt': function() {
            return 'Thumbnail ' + ($(this).index() + 1);
        }
    });
    
    // Form states
    $('.required').prop('required', true);
    $('.optional').prop('required', false);
    
    // Data storage
    $('.product').each(function() {
        $(this).data('price', parseFloat($(this).find('.price').text()));
        $(this).data('inStock', $(this).hasClass('available'));
    });
    
    // Toggle states
    $('.toggle-button').click(function() {
        var isActive = $(this).prop('checked');
        $(this).data('state', isActive ? 'on' : 'off');
    });
});
```

### CSS Manipulation
```javascript
// Get CSS properties
$('#myDiv').css('color');              // Get color
$('#myDiv').css(['color', 'font-size']); // Get multiple properties

// Set CSS properties
$('#myDiv').css('color', 'red');       // Set color
$('#myDiv').css('font-size', '16px');  // Set font size

// Set multiple properties
$('#myDiv').css({
    'color': 'blue',
    'font-size': '18px',
    'background-color': '#f0f0f0',
    'padding': '10px',
    'border': '1px solid #ccc'
});

// CSS classes
$('#myDiv').addClass('highlight');     // Add class
$('#myDiv').removeClass('highlight'); // Remove class
$('#myDiv').toggleClass('active');    // Toggle class
$('#myDiv').hasClass('active');       // Check if has class

// Multiple classes
$('#myDiv').addClass('class1 class2 class3');
$('#myDiv').removeClass('class1 class2');

// Conditional classes
$('.item').addClass(function(index) {
    return 'item-' + index;
});

// Width and height
$('#myDiv').width();                   // Get width (content)
$('#myDiv').innerWidth();              // Get width + padding
$('#myDiv').outerWidth();              // Get width + padding + border
$('#myDiv').outerWidth(true);          // Get width + padding + border + margin

$('#myDiv').height(200);               // Set height
$('#myDiv').width('50%');              // Set width

// Position
$('#myDiv').offset();                  // Get position relative to document
$('#myDiv').position();                // Get position relative to parent

// Scroll position
$(window).scrollTop();                 // Get scroll position
$(window).scrollLeft();                // Get horizontal scroll
$(window).scrollTop(100);              // Set scroll position

// Examples
$(document).ready(function() {
    // Styling navigation
    $('.nav-item').css({
        'display': 'inline-block',
        'padding': '10px 15px',
        'margin': '0 5px',
        'background-color': '#f8f9fa',
        'border-radius': '4px'
    });
    
    // Responsive design helpers
    function updateLayout() {
        var windowWidth = $(window).width();
        
        if (windowWidth < 768) {
            $('.sidebar').addClass('mobile');
            $('.content').css('margin-left', '0');
        } else {
            $('.sidebar').removeClass('mobile');
            $('.content').css('margin-left', '250px');
        }
    }
    
    $(window).resize(updateLayout);
    updateLayout();
    
    // Dynamic styling based on content
    $('.card').each(function() {
        var priority = $(this).data('priority');
        
        switch (priority) {
            case 'high':
                $(this).addClass('border-danger text-danger');
                break;
            case 'medium':
                $(this).addClass('border-warning text-warning');
                break;
            default:
                $(this).addClass('border-secondary');
        }
    });
    
    // Hover effects
    $('.button').hover(
        function() { // Mouse enter
            $(this).addClass('btn-hover');
        },
        function() { // Mouse leave
            $(this).removeClass('btn-hover');
        }
    );
});
```

### Adding and Removing Elements
```javascript
// Creating elements
var newDiv = $('<div>');               // Empty div
var paragraph = $('<p>Hello World</p>'); // Paragraph with text
var list = $('<ul><li>Item 1</li><li>Item 2</li></ul>');

// Adding elements
$('#container').append('<p>Added at end</p>');      // Add as last child
$('#container').prepend('<p>Added at start</p>');   // Add as first child
$('#myDiv').after('<p>Added after</p>');            // Add as next sibling
$('#myDiv').before('<p>Added before</p>');          // Add as previous sibling

// Adding multiple elements
$('#list').append([
    $('<li>Item 1</li>'),
    $('<li>Item 2</li>'),
    $('<li>Item 3</li>')
]);

// Clone elements
var clonedDiv = $('#originalDiv').clone();           // Clone element
var clonedWithEvents = $('#originalDiv').clone(true); // Clone with events
$('#container').append(clonedDiv);

// Moving elements
$('#moveMe').appendTo('#newParent');                 // Move to new parent
$('#moveMe').prependTo('#newParent');                // Move to start of parent

// Removing elements
$('#removeMe').remove();                             // Remove completely
$('#hideMe').detach();                               // Remove but keep data/events
$('#clearMe').empty();                               // Remove all children

// Replace elements
$('#oldElement').replaceWith('<div>New content</div>');
$('<span>Replacement</span>').replaceAll('.oldClass');

// Examples
$(document).ready(function() {
    // Dynamic list generation
    var items = ['Apple', 'Banana', 'Cherry', 'Date'];
    var list = $('<ul>').addClass('fruit-list');
    
    $.each(items, function(index, fruit) {
        var listItem = $('<li>')
            .text(fruit)
            .addClass('fruit-item')
            .data('index', index);
        list.append(listItem);
    });
    
    $('#fruit-container').append(list);
    
    // Dynamic table generation
    var tableData = [
        { name: 'John', age: 30, city: 'New York' },
        { name: 'Jane', age: 25, city: 'Los Angeles' },
        { name: 'Bob', age: 35, city: 'Chicago' }
    ];
    
    var table = $('<table>').addClass('data-table');
    var header = $('<tr>').append(
        $('<th>').text('Name'),
        $('<th>').text('Age'),
        $('<th>').text('City'),
        $('<th>').text('Actions')
    );
    table.append($('<thead>').append(header));
    
    var tbody = $('<tbody>');
    $.each(tableData, function(index, person) {
        var row = $('<tr>')
            .append($('<td>').text(person.name))
            .append($('<td>').text(person.age))
            .append($('<td>').text(person.city))
            .append($('<td>').append(
                $('<button>').text('Edit').addClass('btn-edit'),
                $('<button>').text('Delete').addClass('btn-delete')
            ));
        tbody.append(row);
    });
    
    table.append(tbody);
    $('#table-container').append(table);
    
    // Dynamic form builder
    function createFormField(type, label, name, options = {}) {
        var fieldContainer = $('<div>').addClass('form-field');
        var labelElement = $('<label>').text(label).attr('for', name);
        var input;
        
        switch (type) {
            case 'text':
            case 'email':
            case 'password':
                input = $('<input>').attr({
                    'type': type,
                    'name': name,
                    'id': name
                });
                break;
            case 'textarea':
                input = $('<textarea>').attr({
                    'name': name,
                    'id': name,
                    'rows': options.rows || 3
                });
                break;
            case 'select':
                input = $('<select>').attr({
                    'name': name,
                    'id': name
                });
                if (options.options) {
                    $.each(options.options, function(value, text) {
                        input.append($('<option>').val(value).text(text));
                    });
                }
                break;
        }
        
        if (options.required) {
            input.prop('required', true);
        }
        
        fieldContainer.append(labelElement, input);
        return fieldContainer;
    }
    
    var form = $('<form>').addClass('dynamic-form');
    form.append(
        createFormField('text', 'Name:', 'name', { required: true }),
        createFormField('email', 'Email:', 'email', { required: true }),
        createFormField('select', 'Country:', 'country', {
            options: { 'us': 'United States', 'ca': 'Canada', 'uk': 'United Kingdom' }
        }),
        createFormField('textarea', 'Comments:', 'comments'),
        $('<button>').attr('type', 'submit').text('Submit')
    );
    
    $('#form-container').append(form);
});
```

## Event Handling

### Basic Event Binding
```javascript
// Click events
$('#myButton').click(function() {
    alert('Button clicked!');
});

// Multiple event types
$('#myElement').on('click mouseenter mouseleave', function(event) {
    console.log('Event type:', event.type);
});

// Event with data
$('#myButton').on('click', { message: 'Hello World' }, function(event) {
    alert(event.data.message);
});

// One-time events
$('#myButton').one('click', function() {
    alert('This will only fire once!');
});

// Event delegation (for dynamic elements)
$('#container').on('click', '.dynamic-button', function() {
    console.log('Dynamic button clicked:', $(this).text());
});

// Preventing default behavior
$('a.prevent-default').click(function(event) {
    event.preventDefault();
    console.log('Link click prevented');
});

// Stopping event propagation
$('.child').click(function(event) {
    event.stopPropagation();
    console.log('Child clicked, propagation stopped');
});

// Event object properties
$('#myInput').keyup(function(event) {
    console.log('Key code:', event.which);
    console.log('Key value:', event.target.value);
    console.log('Ctrl pressed:', event.ctrlKey);
    console.log('Mouse position:', event.pageX, event.pageY);
});

// Common events
$(document).ready(function() {
    // Form events
    $('#myForm').submit(function(event) {
        event.preventDefault();
        console.log('Form submitted');
    });
    
    $('#myInput').focus(function() {
        $(this).addClass('focused');
    }).blur(function() {
        $(this).removeClass('focused');
    });
    
    $('#mySelect').change(function() {
        console.log('Selection changed to:', $(this).val());
    });
    
    // Mouse events
    $('#myDiv').mouseenter(function() {
        $(this).addClass('hovered');
    }).mouseleave(function() {
        $(this).removeClass('hovered');
    });
    
    // Keyboard events
    $(document).keydown(function(event) {
        if (event.which === 27) { // Escape key
            $('.modal').hide();
        }
    });
    
    // Window events
    $(window).scroll(function() {
        var scrollTop = $(this).scrollTop();
        $('.scroll-indicator').css('width', (scrollTop / $(document).height() * 100) + '%');
    });
    
    $(window).resize(function() {
        console.log('Window resized to:', $(this).width(), 'x', $(this).height());
    });
});
```

### Advanced Event Handling
```javascript
// Custom events
$(document).ready(function() {
    // Trigger custom event
    $('#myButton').click(function() {
        $(this).trigger('customEvent', ['data1', 'data2']);
    });
    
    // Listen for custom event
    $('#myButton').on('customEvent', function(event, data1, data2) {
        console.log('Custom event fired with data:', data1, data2);
    });
    
    // Event namespacing
    $('#myElement').on('click.namespace1', function() {
        console.log('Namespace 1 click handler');
    });
    
    $('#myElement').on('click.namespace2', function() {
        console.log('Namespace 2 click handler');
    });
    
    // Remove specific namespaced events
    $('#myElement').off('click.namespace1');
    
    // Event delegation with complex selectors
    $('#table').on('click', 'tr:not(.header) td:first-child', function() {
        var row = $(this).closest('tr');
        row.toggleClass('selected');
    });
    
    // Chained event handlers
    $('.interactive')
        .on('mouseenter', function() {
            $(this).addClass('hover');
        })
        .on('mouseleave', function() {
            $(this).removeClass('hover');
        })
        .on('click', function() {
            $(this).toggleClass('active');
        });
    
    // Event handler with context
    var context = {
        name: 'MyApp',
        version: '1.0'
    };
    
    $('#myButton').on('click', context, function(event) {
        console.log('App name:', event.data.name);
        console.log('App version:', event.data.version);
    });
});

// Event delegation examples
$(document).ready(function() {
    // Dynamic list management
    $('#todo-list').on('click', '.todo-item .delete', function() {
        $(this).closest('.todo-item').fadeOut(function() {
            $(this).remove();
        });
    });
    
    $('#todo-list').on('click', '.todo-item .toggle', function() {
        $(this).closest('.todo-item').toggleClass('completed');
    });
    
    $('#add-todo').click(function() {
        var todoText = $('#todo-input').val();
        if (todoText.trim()) {
            var todoItem = $('<div>').addClass('todo-item').html(`
                <span class="text">${todoText}</span>
                <button class="toggle">Toggle</button>
                <button class="delete">Delete</button>
            `);
            $('#todo-list').append(todoItem);
            $('#todo-input').val('');
        }
    });
    
    // Modal system
    $(document).on('click', '[data-modal]', function() {
        var modalId = $(this).data('modal');
        $('#' + modalId).show();
    });
    
    $(document).on('click', '.modal .close', function() {
        $(this).closest('.modal').hide();
    });
    
    $(document).on('click', '.modal-overlay', function() {
        $('.modal').hide();
    });
    
    // Sortable list with drag events
    $('#sortable-list').on('dragstart', '.sortable-item', function(event) {
        event.originalEvent.dataTransfer.setData('text/plain', $(this).index());
    });
    
    $('#sortable-list').on('dragover', '.sortable-item', function(event) {
        event.preventDefault();
    });
    
    $('#sortable-list').on('drop', '.sortable-item', function(event) {
        event.preventDefault();
        var dragIndex = parseInt(event.originalEvent.dataTransfer.getData('text/plain'));
        var dropIndex = $(this).index();
        
        if (dragIndex !== dropIndex) {
            var items = $('#sortable-list .sortable-item');
            var draggedItem = items.eq(dragIndex);
            
            if (dragIndex < dropIndex) {
                draggedItem.insertAfter(items.eq(dropIndex));
            } else {
                draggedItem.insertBefore(items.eq(dropIndex));
            }
        }
    });
});
```

## Effects and Animations

### Basic Effects
```javascript
// Show and hide
$('#myDiv').hide();                    // Hide immediately
$('#myDiv').show();                    // Show immediately
$('#myDiv').toggle();                  // Toggle visibility

// With duration
$('#myDiv').hide(1000);                // Hide over 1 second
$('#myDiv').show('slow');              // Show slowly (600ms)
$('#myDiv').toggle('fast');            // Toggle quickly (200ms)

// With callback
$('#myDiv').hide(500, function() {
    console.log('Hide animation complete');
});

// Fade effects
$('#myDiv').fadeIn();                  // Fade in
$('#myDiv').fadeOut();                 // Fade out
$('#myDiv').fadeToggle();              // Toggle fade
$('#myDiv').fadeTo(1000, 0.5);         // Fade to 50% opacity

// Slide effects
$('#myDiv').slideDown();               // Slide down
$('#myDiv').slideUp();                 // Slide up
$('#myDiv').slideToggle();             // Toggle slide

// Examples
$(document).ready(function() {
    // Dropdown menu
    $('.dropdown-trigger').click(function() {
        $(this).next('.dropdown-menu').slideToggle(300);
    });
    
    // Image gallery
    $('.thumbnail').click(function() {
        var largeSrc = $(this).data('large');
        $('#lightbox-img').attr('src', largeSrc);
        $('#lightbox').fadeIn(400);
    });
    
    $('.lightbox-close').click(function() {
        $('#lightbox').fadeOut(400);
    });
    
    // Notification system
    function showNotification(message, type = 'info') {
        var notification = $('<div>')
            .addClass('notification notification-' + type)
            .text(message)
            .hide();
        
        $('#notifications').append(notification);
        notification.slideDown(300);
        
        setTimeout(function() {
            notification.slideUp(300, function() {
                $(this).remove();
            });
        }, 3000);
    }
    
    // Accordion
    $('.accordion-header').click(function() {
        var content = $(this).next('.accordion-content');
        var isOpen = content.is(':visible');
        
        $('.accordion-content').slideUp(300);
        
        if (!isOpen) {
            content.slideDown(300);
        }
    });
});
```

### Custom Animations
```javascript
// animate() method
$('#myDiv').animate({
    left: '250px',
    opacity: 0.5,
    height: 'toggle',
    width: 'toggle'
}, 1000);

// Multiple animations in sequence
$('#myDiv')
    .animate({ left: '250px' }, 500)
    .animate({ top: '100px' }, 500)
    .animate({ opacity: 0.5 }, 500);

// Animation with easing
$('#myDiv').animate({
    left: '250px'
}, {
    duration: 1000,
    easing: 'swing', // or 'linear'
    complete: function() {
        console.log('Animation complete');
    },
    progress: function(animation, progress, remainingMs) {
        console.log('Progress:', Math.round(progress * 100) + '%');
    }
});

// Stop animations
$('#myDiv').stop();                    // Stop current animation
$('#myDiv').stop(true);                // Stop and clear queue
$('#myDiv').stop(true, true);          // Stop, clear queue, and jump to end

// Animation queue management
$('#myDiv').queue(function() {
    $(this).addClass('highlighted');
    $(this).dequeue(); // Continue to next animation
});

// Complex animation examples
$(document).ready(function() {
    // Bouncing ball animation
    function bounceBall() {
        $('#ball')
            .animate({ top: '300px' }, 1000, 'linear')
            .animate({ top: '50px' }, 800, 'linear')
            .animate({ top: '250px' }, 600, 'linear')
            .animate({ top: '100px' }, 400, 'linear')
            .animate({ top: '200px' }, 200, 'linear', bounceBall);
    }
    
    $('#start-bounce').click(bounceBall);
    
    // Loading spinner
    function startSpinner() {
        $('#spinner').animate({ rotate: '360deg' }, 1000, 'linear', startSpinner);
    }
    
    // Progress bar animation
    function animateProgressBar(percentage) {
        $('#progress-bar').animate({
            width: percentage + '%'
        }, {
            duration: 2000,
            step: function(now) {
                $('#progress-text').text(Math.round(now) + '%');
            },
            complete: function() {
                $('#progress-text').text('Complete!');
            }
        });
    }
    
    // Card flip animation
    $('.card').click(function() {
        var card = $(this);
        
        card.animate({ rotateY: '90deg' }, 300, function() {
            card.find('.front').hide();
            card.find('.back').show();
            card.animate({ rotateY: '0deg' }, 300);
        });
    });
    
    // Typewriter effect
    function typeWriter(element, text, speed = 100) {
        var i = 0;
        element.empty();
        
        function type() {
            if (i < text.length) {
                element.text(element.text() + text.charAt(i));
                i++;
                setTimeout(type, speed);
            }
        }
        
        type();
    }
    
    // Parallax scrolling effect
    $(window).scroll(function() {
        var scrollTop = $(this).scrollTop();
        
        $('.parallax-bg').css('transform', 'translateY(' + (scrollTop * 0.5) + 'px)');
        $('.parallax-content').css('transform', 'translateY(' + (scrollTop * 0.2) + 'px)');
    });
    
    // Count up animation
    function countUp(element, start, end, duration) {
        var startTime = null;
        
        function step(timestamp) {
            if (!startTime) startTime = timestamp;
            var progress = Math.min((timestamp - startTime) / duration, 1);
            var current = Math.floor(progress * (end - start) + start);
            
            element.text(current);
            
            if (progress < 1) {
                requestAnimationFrame(step);
            }
        }
        
        requestAnimationFrame(step);
    }
    
    // Trigger count up when element comes into view
    $(window).scroll(function() {
        $('.counter').each(function() {
            var elementTop = $(this).offset().top;
            var viewportTop = $(window).scrollTop();
            var viewportBottom = viewportTop + $(window).height();
            
            if (elementTop < viewportBottom && !$(this).hasClass('counted')) {
                $(this).addClass('counted');
                var target = parseInt($(this).data('target'));
                countUp($(this), 0, target, 2000);
            }
        });
    });
});
```

## AJAX with jQuery

### Basic AJAX Requests
```javascript
// GET request
$.get('api/users', function(data) {
    console.log('Users:', data);
}).fail(function() {
    console.error('Failed to load users');
});

// POST request
$.post('api/users', {
    name: 'John Doe',
    email: 'john@example.com'
}, function(data) {
    console.log('User created:', data);
});

// Generic AJAX request
$.ajax({
    url: 'api/users/123',
    method: 'GET',
    dataType: 'json',
    success: function(data) {
        console.log('User data:', data);
    },
    error: function(xhr, status, error) {
        console.error('Error:', error);
    }
});

// AJAX with all options
$.ajax({
    url: 'api/data',
    method: 'POST',
    data: JSON.stringify({ key: 'value' }),
    contentType: 'application/json',
    dataType: 'json',
    timeout: 5000,
    headers: {
        'Authorization': 'Bearer token123'
    },
    beforeSend: function(xhr) {
        $('#loading').show();
    },
    complete: function() {
        $('#loading').hide();
    },
    success: function(data, textStatus, xhr) {
        console.log('Success:', data);
    },
    error: function(xhr, textStatus, errorThrown) {
        console.error('Error:', errorThrown);
    }
});

// JSON convenience method
$.getJSON('api/config.json', function(config) {
    console.log('Config loaded:', config);
});

// Load HTML content
$('#content').load('pages/about.html', function(responseText, textStatus) {
    if (textStatus === 'success') {
        console.log('Content loaded successfully');
    }
});

// Load specific element from external page
$('#sidebar').load('pages/menu.html #navigation');
```

### Advanced AJAX Patterns
```javascript
// AJAX with promises (jQuery 1.5+)
$.ajax({
    url: 'api/users',
    method: 'GET'
})
.done(function(data) {
    console.log('Success:', data);
})
.fail(function(xhr, status, error) {
    console.error('Error:', error);
})
.always(function() {
    console.log('Request completed');
});

// Chaining AJAX requests
$.get('api/user/123')
    .then(function(user) {
        return $.get('api/user/' + user.id + '/posts');
    })
    .then(function(posts) {
        console.log('User posts:', posts);
    })
    .catch(function(error) {
        console.error('Chain failed:', error);
    });

// Multiple concurrent requests
$.when(
    $.get('api/users'),
    $.get('api/posts'),
    $.get('api/comments')
).done(function(users, posts, comments) {
    console.log('All data loaded:', {
        users: users[0],
        posts: posts[0],
        comments: comments[0]
    });
}).fail(function() {
    console.error('One or more requests failed');
});

// Global AJAX settings
$.ajaxSetup({
    timeout: 10000,
    headers: {
        'X-Requested-With': 'XMLHttpRequest'
    },
    beforeSend: function() {
        NProgress.start(); // Progress bar library
    },
    complete: function() {
        NProgress.done();
    }
});

// AJAX error handling
$(document).ajaxError(function(event, xhr, settings, error) {
    console.error('AJAX Error:', {
        url: settings.url,
        status: xhr.status,
        error: error
    });
    
    if (xhr.status === 401) {
        window.location.href = '/login';
    }
});

// Practical examples
$(document).ready(function() {
    // Auto-save form
    $('#auto-save-form input, #auto-save-form textarea').on('input', function() {
        clearTimeout(window.autoSaveTimeout);
        window.autoSaveTimeout = setTimeout(function() {
            var formData = $('#auto-save-form').serialize();
            
            $.post('api/auto-save', formData)
                .done(function() {
                    $('#save-status').text('Saved').addClass('success');
                })
                .fail(function() {
                    $('#save-status').text('Save failed').addClass('error');
                });
        }, 2000);
    });
    
    // Infinite scroll
    $(window).scroll(function() {
        if ($(window).scrollTop() + $(window).height() >= $(document).height() - 100) {
            loadMoreContent();
        }
    });
    
    var currentPage = 1;
    var loading = false;
    
    function loadMoreContent() {
        if (loading) return;
        loading = true;
        
        $.get('api/posts', { page: currentPage + 1 })
            .done(function(posts) {
                if (posts.length > 0) {
                    $('#content').append(posts.map(createPostElement));
                    currentPage++;
                }
                loading = false;
            })
            .fail(function() {
                loading = false;
            });
    }
    
    // Real-time search
    $('#search-input').on('input', function() {
        var query = $(this).val();
        
        if (query.length >= 3) {
            clearTimeout(window.searchTimeout);
            window.searchTimeout = setTimeout(function() {
                $.get('api/search', { q: query })
                    .done(function(results) {
                        displaySearchResults(results);
                    });
            }, 300);
        } else {
            $('#search-results').empty();
        }
    });
    
    // File upload with progress
    $('#file-upload-form').submit(function(event) {
        event.preventDefault();
        
        var formData = new FormData(this);
        
        $.ajax({
            url: 'api/upload',
            type: 'POST',
            data: formData,
            processData: false,
            contentType: false,
            xhr: function() {
                var xhr = new window.XMLHttpRequest();
                xhr.upload.addEventListener('progress', function(event) {
                    if (event.lengthComputable) {
                        var percentComplete = (event.loaded / event.total) * 100;
                        $('#upload-progress').css('width', percentComplete + '%');
                    }
                });
                return xhr;
            },
            success: function(response) {
                console.log('Upload successful:', response);
            },
            error: function() {
                console.error('Upload failed');
            }
        });
    });
});
```

---

## Related Topics
- [[11 - DOM Manipulation]]
- [[17 - Event Methods]]
- [[21 - XMLHttpRequest]]
- [[22 - Fetch API]]
- [[32 - Frontend Frameworks]]

---

*Next: [[32 - Frontend Frameworks]]*
