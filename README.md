# digital-book<!doctype html>
<html lang="en">
<head>
	<meta name="viewport" content="width=device-width, initial-scale=1">
	<meta charset="UTF-8">
	<link rel="stylesheet" href="../book/css/style2.css">
	<title>Digital-Book</title>
	<meta name="digital book" content="digital book">
	<meta name="keywords" content="camino de santiago levante">
   <script type="text/javascript" src="../../extras/jquery.min.1.7.js"></script>
   <script type="text/javascript" src="../../extras/jquery-ui-1.8.20.custom.min.js"></script>
   <script type="text/javascript" src="../../extras/jquery.mousewheel.min.js"></script>
   <script type="text/javascript" src="../../extras/modernizr.2.5.3.min.js"></script>
   <script type="text/javascript" src="../../lib/hash.js"></script>
</head>
<body>
<div id="canvas">
	<div id="book-zoom">
		<div class="sj-book">
			<div depth="5" class="hard"> <div class="side"></div> </div>
			<div depth="5" class="hard front-side"> <div class="depth"></div> </div>
			<div class="own-size"></div>
			<div class="own-size even"></div>
			<div class="hard fixed back-side p67"> <div class="depth"></div> </div>
			<div class="hard p68"></div>
		</div>
	</div>
	<div id="slider-bar" class="turnjs-slider">
		<div id="slider"></div>
	</div>
</div>
<script type="text/javascript">
function loadApp() {
	
	var flipbook = $('.sj-book');

	
	if (flipbook.width()==0 || flipbook.height()==0) {
		setTimeout(loadApp, 10);
		return;
	}
	$('#book-zoom').mousewheel(function(event, delta, deltaX, deltaY) {

		var data = $(this).data(),
			step = 30,
			flipbook = $('.sj-book'),
			actualPos = $('#slider').slider('value')*step;

		if (typeof(data.scrollX)==='undefined') {
			data.scrollX = actualPos;
			data.scrollPage = flipbook.turn('page');
		}

		data.scrollX = Math.min($( "#slider" ).slider('option', 'max')*step,
			Math.max(0, data.scrollX + deltaX));

		var actualView = Math.round(data.scrollX/step),
			page = Math.min(flipbook.turn('pages'), Math.max(1, actualView*2 - 2));

		if ($.inArray(data.scrollPage, flipbook.turn('view', page))==-1) {
			data.scrollPage = page;
			flipbook.turn('page', page);
		}

		if (data.scrollTimer)
			clearInterval(data.scrollTimer);
		
		data.scrollTimer = setTimeout(function(){
			data.scrollX = undefined;
			data.scrollPage = undefined;
			data.scrollTimer = undefined;
		}, 1000);

	});
	$( "#slider" ).slider({
		min: 1,
		max: 100,

		start: function(event, ui) {

			if (!window._thumbPreview) {
				_thumbPreview = $('<div />', {'class': 'thumbnail'}).html('<div></div>');
				setPreview(ui.value);
				_thumbPreview.appendTo($(ui.handle));
			} else
				setPreview(ui.value);

			moveBar(false);

		},

		slide: function(event, ui) {

			setPreview(ui.value);

		},

		stop: function() {

			if (window._thumbPreview)
				_thumbPreview.removeClass('show');
			
			$('.sj-book').turn('page', Math.max(1, $(this).slider('value')*2 - 2));

		}
	});
	Hash.on('^page\/([0-9]*)$', {
		yep: function(path, parts) {

			var page = parts[1];

			if (page!==undefined) {
				if ($('.sj-book').turn('is'))
					$('.sj-book').turn('page', page);
			}

		},
		nop: function(path) {

			if ($('.sj-book').turn('is'))
				$('.sj-book').turn('page', 1);
		}
	});
	$(document).keydown(function(e){

		var previous = 37, next = 39;

		switch (e.keyCode) {
			case previous:

				$('.sj-book').turn('previous');

			break;
			case next:
				
				$('.sj-book').turn('next');

			break;
		}

	});
	flipbook.bind(($.isTouch) ? 'touchend' : 'click', zoomHandle);

	flipbook.turn({
		elevation: 50,
		acceleration: !isChrome(),
		autoCenter: true,
		gradients: true,
		duration: 1000,
		pages: 68,
		when: {
			turning: function(e, page, view) {
				
				var book = $(this),
					currentPage = book.turn('page'),
					pages = book.turn('pages');

				if (currentPage>3 && currentPage<pages-3) {
				
					if (page==1) {
						book.turn('page', 2).turn('stop').turn('page', page);
						e.preventDefault();
						return;
					} else if (page==pages) {
						book.turn('page', pages-1).turn('stop').turn('page', page);
						e.preventDefault();
						return;
					}
				} else if (page>3 && page<pages-3) {
					if (currentPage==1) {
						book.turn('page', 2).turn('stop').turn('page', page);
						e.preventDefault();
						return;
					} else if (currentPage==pages) {
						book.turn('page', pages-1).turn('stop').turn('page', page);
						e.preventDefault();
						return;
					}
				}

				updateDepth(book, page);
				
				if (page>=2)
					$('.sj-book .p2').addClass('fixed');
				else
					$('.sj-book .p2').removeClass('fixed');

				if (page<book.turn('pages'))
					$('.sj-book .p67').addClass('fixed');
				else
					$('.sj-book .p67').removeClass('fixed');

				Hash.go('page/'+page).update();
					
			},

			turned: function(e, page, view) {

				var book = $(this);

				if (page==2 || page==3) {
					book.turn('peel', 'br');
				}

				updateDepth(book);
				
				$('#slider').slider('value', getViewNumber(book, page));

				book.turn('center');

			},

			start: function(e, pageObj) {
		
				moveBar(true);

			},

			end: function(e, pageObj) {
			
				var book = $(this);

				updateDepth(book);

				setTimeout(function() {
					
					$('#slider').slider('value', getViewNumber(book));

				}, 1);

				moveBar(false);

			},

			missing: function (e, pages) {

				for (var i = 0; i < pages.length; i++) {
					addPage(pages[i], $(this));
				}

			}
		}
	});
	$('#slider').slider('option', 'max', numberOfViews(flipbook));

	flipbook.addClass('animated');
	$('#canvas').css({visibility: ''});
}
$('#canvas').css({visibility: 'hidden'});
yepnope({
	test : Modernizr.csstransforms,
	yep: ['../../lib/turn.min.js'],
	nope: ['../../lib/turn.html4.min.js', 'css/jquery.ui.html4.css', 'css/book-html4.css'],
	both: ['js/book.js', 'css/jquery.ui.css', 'css/book.css'],
	complete: loadApp
});
</script>
</body>
<footer>
	<menu>
		<a class="menu" href="index.html#page/1"> <img src="https://drive.google.com/uc?export=view&id=1CQotjBJjqCD7BPg7OthvBPR4eQ7pO5TD" height="100%" width="50%" alt=""> </a>
		<a class="menu1" href="index.html#page/63"> <img src="https://drive.google.com/uc?export=view&id=1eQfVjh4INCbhhcImd-lIsfr5kYd27Zhz" height="100%" width="40%" alt=""></a>
	</menu>
</footer>
</html>
