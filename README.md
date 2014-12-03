describinggraph
===============

activity.js
============

var mainParent, container, controlPanel, graphHolder, inputBoxes;
var graph, numericKeyPad;
var feedback, buttons;
var blocker, promptBox, settingsPanel, errorBox;
var isAnimating = false, aniTime = 500;
var promptFunc = {yes:null, no:null, parameters:{}};
var lastColorPicker = null;
var eventType = "click";
var tagCount= 7;
var tagArray = ["increasing","decreasing","constant","maximum","minimum","linear","non-linear"];
var positions = [];
var canDrag = true;
var prevDeg = 0;
var dragHoldX,dragHoldY,leftDiff,topDiff,indexVal = 1
var boundaryLeft,boundaryTop,boundaryRight,boundaryBottom
$(document).ready(function(){
    if ("ontouchstart" in document.documentElement) eventType = "touchstart";
    mainParent = $("body");
    var table=["<div id=\"blocker\" onclick=\"instruct()\"></div><div class=\"question\" onclick=\"instruct()\"><div style=\"position:absolute; left:2%; top:5%; opacity:0.6;\" id=\"instructions-button\"><img src=\"images/btInfoOn.png\" width=\"25px\" height=\"27px\" /></div><span style=\"text-align:left; font-family:myFontfamily; font-size:24px;left:40px; line-height:1.4em; width:390px; position:absolute; top:45px;\">Use as many tags as you need to describe all the parts of the graph.</span><div class=\"closeBtn\" >&#x000D7;</div><div style=\"position:absolute; left:43%; top:75%;\" ></div></div>"];
    mainParent.bind("touchmove", function(e){ e.preventDefault(); });
    container = $("<div id=\"container\" class=\"container\"></div>");
    controlPanel = $("<div id=\"controlPanel\"><div class=\"taghead\">Tags</div></div>");
    graphHolder = $("<div id=\"graphHolder\"></div>");
    graphContainer = $("<div id=\"graphContainer\"></div>");
    activityHolder = $("<div id=\"activityHolder\"></div>");
    dragHolder = $("<div id=\"dragHolder\"></div>");
    blocker = $("<div id=\"generalBlocker\" style=\"z-index:100000;\" onclick=\"instruct()\"></div>");
    feedback = $("<div class=contentFeedback> <div class=btReset><div class=icoReset></div></div> <div class=\"btInfo\"><div class=\"icoInfo\" onclick=\"openInst()\"></div></div><div class=\"btFeedback\"> <div class=\"icoFeed\"></div> </div> <div class=\"score\"> <div id=\"contCorrect\">0 <img src=\"images/IconCorrect.png\"></div><div id=\"contIncorrect\">0 <img src=\"images/IconIncorrect.png\"></div></div></div>");
    container.append(table[0]);
    container.append(activityHolder).append(dragHolder).append(feedback).append(blocker);
    activityHolder.append(graphContainer).append(graphHolder).append(controlPanel);
    mainParent.append(container);
    container.append("<div id=\"tempTitle\" class=\"title\"></div>");
    blocker.show();
    container.hide();
    $(".icoReset").css({"opacity":"0.5"});
    $(".icoReset").bind(eventType, resetClick);
    $(".icoReset").bind('click',resetClick);
    graph = new Graph(graphHolder, {});
    graph.sketchMode = "";
    setTimeout(function(){
	container.show();
	$("body").css({"-webkit-transform":"scale(1)"});
	eventBroker = _({}).extend(require('chaplin/lib/event_broker'));
	eventBroker.publishEvent("#fetch", { type : 'state' }, function(state) {
	    if (state) {
		_.each(JSON.parse(state), function(value, key, list) {
		    if (key == "indexVal") {
		       indexVal = value
		    }
		    if (key == "graphContainer") {
			$("#graphContainer").html(value);
			$("#graphContainer .draggable_wp").bind("mousedown touchstart", function(event){
			    var pos = {x: $(this).offset().left, y: $(this).offset().top};
			    var index = -1;
			    for(var i=0; i<positions.length; i++){
				if (positions[i].element == this) {
				    index = i;
				    break;
				}
			    }
			    if (index < 0) positions.push({element:this, pos:pos});
			    else positions[index] = {element:this, pos:pos};
			});
			$("#graphContainer .draggable_wp").draggable({
			    start:function(event,ui) {
				indexVal++;
				$(this).css({'z-index':indexVal})
				target_wp = $(this).find('.rotatable');
				var rect = target_wp[0].getBoundingClientRect()
				degree = target_wp.attr("style");
				if (degree.indexOf("rotate") >= 0) {
				    degree = degree.substr(degree.indexOf("rotate")+7);
				    degree = degree.substr(0, degree.indexOf("deg)"));
				    degree = parseFloat(degree);
				}
				else degree = 0;
				width =target_wp.width()+2
				height = target_wp.height()+2
				var angle = degree * Math.PI / 180,
				sin   = Math.sin(angle),
				cos   = Math.cos(angle);
		    
				var x1 = cos * width,
				    y1 = sin * width;
				
				var x2 = -sin * height,
				    y2 = cos * height;
				
				var x3 = cos * width - sin * height,
				    y3 = sin * width + cos * height;
				
				var minX = Math.floor(Math.min(0, x1, x2, x3)),
				maxX = Math.floor(Math.max(0, x1, x2, x3)),
				minY = Math.floor(Math.min(0, y1, y2, y3)),
				maxY = Math.floor(Math.max(0, y1, y2, y3));
				
				if (degree>60 && degree<=90) {
				    maxY+=10
				}
				if (degree>90 && degree<135) {
				    maxY+=18
				}
				if (degree>=135 && degree<180) {
				    maxY+=24
				}
				if (maxY<height) {
				    maxY = height
				}
				if (degree==0) {
				    minY = -2
				}
				
				boundaryLeft = ((rect.left-$(this).offset().left)*-1)+2
				boundaryTop = (minY*-1)
				boundaryBottom = ($('#graphContainer').height())-maxY
				boundaryRight = ($('#dragHolder').width())-maxX
				boundaryRight_1 = ($('#graphContainer').width())-maxX
				
				dragHoldX = event.pageX-$(this).offset().left
				dragHoldY = event.pageY-$(this).offset().top
			    },
			    drag:function(event,ui) {
				leftPos = event.pageX-($('#graphContainer').offset().left+3)-dragHoldX
				topPos = event.pageY-($('#graphContainer').offset().top+3)-dragHoldY
				if(leftPos<boundaryLeft) {
				    leftPos = boundaryLeft
				}
				if(leftPos>boundaryRight) {
				    leftPos = boundaryRight
				}
				if(topPos<boundaryTop) {
				    topPos = boundaryTop
				}
				if(topPos>boundaryBottom) {
				    topPos = boundaryBottom
				}
				ui.position.left = leftPos
				ui.position.top = topPos
			    },
			    stop:function(event,ui){
				leftPos = event.pageX-($('#graphContainer').offset().left+3)-dragHoldX
				topPos = event.pageY-($('#graphContainer').offset().top+3)-dragHoldY
				if(leftPos>boundaryRight_1) {
				    $(this).remove();
				}
				else {
				    var pos;
				    var index = -1;
				    for(var i=0; i<positions.length; i++){
					if (positions[i].element == this) {
					    index = i;
					    break;
					}
				    }
				    if (index >= 0) pos = positions[index];
				    if (pos) {		
					var target_wp = ui.helper.eq(0);
					var variationX = pos.pos.x - target_wp.offset().left;
					var variationY = pos.pos.y - target_wp.offset().top;
					var origin = target_wp.data("origin");
					if (target_wp.data("origin")) {
					    target_wp.data("origin", {
						left: origin.left-variationX,
						top: origin.top-variationY
					    });
					}
				    }
				}    
			    }
			});
			rotateDiv();
		    }
		    if (key == "resetStatus") {
		       $('.icoReset').css('opacity',value);
		    }
		});
	    }
	});
	graph.drawGraph(false,0,0,true);
	LoadTiles();
	ApplyDraggables();
	$("body").css({"-webkit-transform":"scale(1)"});
    }, 1000);
});
function LoadTiles(){
    for(var tag=0; tag<tagCount; tag++)
    {
	controlPanel.append('<div class="draggable_wp" id="tag'+tag+'"><div class="rotatable" style=""><div class="rotatable1"></div><div class="rotatable2"><div class="el">'+tagArray[tag]+'</div><div class="handle"></div></div></div></div>');
    }
    $(".draggable_wp").mousedown(function(){enableReset1();})
}
function ApplyDraggables(){
    boundaryLeft = $("#dragHolder").offset().left
    boundaryTop= $("#dragHolder").offset().top
    boundaryRight = $("#dragHolder").offset().left+$("#dragHolder").width()
    boundaryBottom = $("#dragHolder").offset().top+$("#dragHolder").height()
    
    $('#controlPanel .draggable_wp').draggable({
	containment:"#dragHolder",
	helper:"clone",
	revert:"invalid",
	zIndex: 10000
    });  
    $('#graphContainer ').droppable({
	accept: ".draggable_wp",
	tolerance:"fit",
	drop:function (event, ui){
	    try {
		touchX = (event.originalEvent.changedTouches[0].pageX - $("#graphContainer").offset().left-3);
		touchY = (event.originalEvent.changedTouches[0].pageY - $("#graphContainer").offset().top-3);
	    } catch(e){
		touchX = (event.pageX - $("#graphContainer").offset().left-3);
		touchY = (event.pageY - $("#graphContainer").offset().top-3);
	    }
	    try {
		touchXDraggable = (event.originalEvent.changedTouches[0].pageX - $(ui.helper).offset().left);
		touchYDraggable = (event.originalEvent.changedTouches[0].pageY - $(ui.helper).offset().top);
	    } catch(e){
		touchXDraggable = (event.pageX - $(ui.helper).offset().left);
		touchYDraggable = (event.pageY - $(ui.helper).offset().top);
	    }
	    touchX = touchX - touchXDraggable;
	    touchY = touchY - touchYDraggable;
            var child;
	    indexVal++;

	    if(!$(ui.helper).hasClass("cloned"))
	    child= $(ui.helper).clone();
	    $(child).addClass("cloned");
	    $(child).appendTo($('#graphContainer'));
	    $(child).css({left:(touchX)+"px", top:(touchY)+"px",'z-index':indexVal})
	    $(child).bind("mousedown touchstart", function(event){
		var pos = {x: $(this).offset().left, y: $(this).offset().top};
		var index = -1;
		for(var i=0; i<positions.length; i++){
		    if (positions[i].element == this) {
			index = i;
			break;
		    }
		}
		if (index < 0) positions.push({element:this, pos:pos});
		else positions[index] = {element:this, pos:pos};
	    });
	   
	    $(child).draggable({
		start:function(event,ui) {		    
		    indexVal++;
		    $(this).css({'z-index':indexVal});
		    target_wp = $(this).find('.rotatable');
		    var rect = target_wp[0].getBoundingClientRect()
		    degree = target_wp.attr("style");
		    if (degree.indexOf("rotate") >= 0) {
			degree = degree.substr(degree.indexOf("rotate")+7);
			degree = degree.substr(0, degree.indexOf("deg)"));
			degree = parseFloat(degree);
		    }
		    else degree = 0;
		    width =target_wp.width()+2
		    height = target_wp.height()+2
		    var angle = degree * Math.PI / 180,
		    sin   = Math.sin(angle),
		    cos   = Math.cos(angle);
	
		    var x1 = cos * width,
			y1 = sin * width;
		    
		    var x2 = -sin * height,
			y2 = cos * height;
		    
		    var x3 = cos * width - sin * height,
			y3 = sin * width + cos * height;
		    
		    var minX = Math.floor(Math.min(0, x1, x2, x3)),
		    maxX = Math.floor(Math.max(0, x1, x2, x3)),
		    minY = Math.floor(Math.min(0, y1, y2, y3)),
		    maxY = Math.floor(Math.max(0, y1, y2, y3));
		    
		    if (degree>60 && degree<=90) {
			maxY+=10
		    }
		    if (degree>90 && degree<135) {
			maxY+=18
		    }
		    if (degree>=135 && degree<180) {
			maxY+=24
		    }
		    if (maxY<height) {
			maxY = height
		    }
		    if (degree==0) {
			minY = -2
		    }
		    
		    boundaryLeft = ((rect.left-$(this).offset().left)*-1)+2
		    boundaryTop = (minY*-1)
		    boundaryBottom = ($('#graphContainer').height())-maxY
		    boundaryRight = ($('#dragHolder').width())-maxX
		    boundaryRight_1 = ($('#graphContainer').width())-maxX
		    
		    dragHoldX = event.pageX-$(this).offset().left
		    dragHoldY = event.pageY-$(this).offset().top
		},
		drag:function(event,ui) {		    
		    leftPos = event.pageX-($('#graphContainer').offset().left+3)-dragHoldX
		    topPos = event.pageY-($('#graphContainer').offset().top+3)-dragHoldY
		    if(leftPos<boundaryLeft) {
			leftPos = boundaryLeft
		    }
		    if(leftPos>boundaryRight) {
			leftPos = boundaryRight
		    }
		    if(topPos<boundaryTop) {
			topPos = boundaryTop
		    }
		    if(topPos>boundaryBottom) {
			topPos = boundaryBottom
		    }
		    ui.position.left = leftPos
		    ui.position.top = topPos
		},
		stop:function(event,ui){
		    leftPos = event.pageX-($('#graphContainer').offset().left+3)-dragHoldX
		    topPos = event.pageY-($('#graphContainer').offset().top+3)-dragHoldY
		    if(leftPos>boundaryRight_1) {
			$(this).remove();
		    }
		    else {
			var pos;
			var index = -1;
			for(var i=0; i<positions.length; i++){
			    if (positions[i].element == this) {
				index = i;
				break;
			    }
			}
			if (index >= 0) pos = positions[index];
			if (pos) {		
			    var target_wp = ui.helper.eq(0);
			    var variationX = pos.pos.x - target_wp.offset().left;
			    var variationY = pos.pos.y - target_wp.offset().top;
			    var origin = target_wp.data("origin");
			    if (target_wp.data("origin")) {
				target_wp.data("origin", {
				    left: origin.left-variationX,
				    top: origin.top-variationY
				});
			    }
			}
		    }    
		}
	    });
	    rotateDiv();
	}
    });
    
}
function enableReset1() {
    $(".icoReset").css({"opacity":"1","cursor":"pointer"});
}
function resetClick() {
    if ($('.icoReset').css('opacity') == "1") {
	$(".icoReset").removeClass('icoResetMove');
	graph.reset();
	var to=setTimeout(function(){
	    clearInterval(to);
	    $(".icoReset").addClass('icoResetMove');
	    $('.icoReset').css({'opacity':'0.5','cursor':'default'});
	},200);
	var tagremove = $('#graphContainer').find('div').hasClass('ui-draggable');
	if (tagremove) $('#graphContainer').find('div').remove();
    }
}
function instruct(){
	blocker.hide();
	$('.question,#blocker').hide();
	$('.icoInfo').css({'opacity':'1'});
	try {
		$('#instructions-button-container').unbind('click',openInst);
	} catch(e){}
		$('#instructions-button-container').bind('click',openInst);
}
function openInst(){
	blocker.show();
	$('.question,#blocker').show();
	$('.icoInfo').css({'opacity':'0.5'});
}
function toggleClassButton(element,className){
	var currentButton=element;
	if(!currentButton.hasClass(className)){
		currentButton.addClass(className);
	}else{
		currentButton.removeClass(className);	
	}
}
function rotateDiv() {
    var dragging = false,
    target_wp,
    o_x, o_y, h_x, h_y, last_angle;
    var oldDegree = 0;
    $('.handle').mousedown(function (e) {
	indexVal++;
	$(this).parent().parent().parent().css({'z-index':indexVal});
	canDrag = false;
	var rotapply = $(e.target).parent().parent().parent().hasClass("cloned");
	if (rotapply) {
	    h_x = e.pageX;
	    h_y = e.pageY;
	    e.preventDefault();
	    e.stopPropagation();
	    dragging = true;
	    target_wp = $(e.target).closest('.draggable_wp').find('.rotatable');
	    last_angle = target_wp.data("last_angle") || 0;
	    oldDegree = target_wp.attr("style");
	    if (oldDegree.indexOf("rotate") >= 0) {
		oldDegree = oldDegree.substr(oldDegree.indexOf("rotate")+7);
		oldDegree = oldDegree.substr(0, oldDegree.indexOf("deg)"));
		oldDegree = parseFloat(oldDegree);
	    }
	    else oldDegree = 0;
	    var off_x = target_wp.parent().offset().left;//target_wp.offset().left;
	    var off_y = target_wp.parent().offset().top;//target_wp.offset().top;
	    if (!target_wp.data("origin"))
		target_wp.data("origin", {
		    left: off_x,
		    top: off_y
		});
		o_x = off_x;//target_wp.data("origin").left;
		o_y = off_y;//target_wp.data("origin").top;
	    prevDeg = oldDegree
	}
    })
    $(document).mousemove(function (e) {
        if (dragging) {
            var s_x = e.pageX, s_y = e.pageY; // start rotate point
            if (s_x !== o_x && s_y !== o_y) { //start rotate
                var s_rad = Math.atan2(s_y - o_y, s_x - o_x); // current to origin
                s_rad -= Math.atan2(h_y - o_y, h_x - o_x); // handle to origin
                s_rad += last_angle; // relative to the last one
                var degree = (s_rad * (360 / (2 * Math.PI)))+oldDegree;
		degree = (degree < 0 ? 360 + degree : degree)
		//console.log(degree,oldDegree)
		
		width =target_wp.width()
		height = target_wp.height()
		var angle = degree * Math.PI / 180,
		sin   = Math.sin(angle),
		cos   = Math.cos(angle);
	    
		// (0,0) stays as (0, 0)
		
		// (w,0) rotation
		var x1 = cos * width,
		    y1 = sin * width;
		
		// (0,h) rotation
		var x2 = -sin * height,
		    y2 = cos * height;
		
		// (w,h) rotation
		var x3 = cos * width - sin * height,
		    y3 = sin * width + cos * height;
		
		var minX = Math.min(0, x1, x2, x3),
		    maxX = Math.max(0, x1, x2, x3),
		    minY = Math.min(0, y1, y2, y3),
		    maxY = Math.max(0, y1, y2, y3);
		
		var rotatedWidth  = maxX - minX, rotatedHeight = maxY - minY;

		if (degree >= 90 && degree <= 270) {
		    target_wp.find('.el').css({'-webkit-transform':'rotate(180deg)'});
		}
		else {
		    target_wp.find('.el').css({'-webkit-transform':'rotate(0deg)'});
		}
		if (degree>360) {
		    degree-=360
		}
		target_wp.css('-moz-transform', 'rotate(' + degree + 'deg)');
		target_wp.css('-moz-transform-origin', '0% 50%');
		target_wp.css('-webkit-transform', 'rotate(' + degree + 'deg)');
		target_wp.css('-webkit-transform-origin', '0% 50%');
		target_wp.css('-o-transform', 'rotate(' + degree + 'deg)');
		target_wp.css('-o-transform-origin', '0% 50%');
		target_wp.css('-ms-transform', 'rotate(' + degree + 'deg)');
		target_wp.css('-ms-transform-origin', '0% 50%');
		var rect = target_wp[0].getBoundingClientRect()
		if (degree>90 & degree<180) {
		    maxY+=25
		}
		if (rect.left<$('#graphContainer').offset().left+3 || rect.top<$('#graphContainer').offset().top+3 || rect.left+rect.width>$('#graphContainer').offset().left+$('#graphContainer').width()-6 || maxY+target_wp.parent().offset().top>$('#graphContainer').offset().top+$('#graphContainer').height()-6) {
		    degree = prevDeg
		}
		if (degree >= 90 && degree <= 270) {
		    target_wp.find('.el').css({'-webkit-transform':'rotate(180deg)'});
		}
		else {
		    target_wp.find('.el').css({'-webkit-transform':'rotate(0deg)'});
		}
		if (degree>360) {
		    degree-=360
		}
		target_wp.css('-moz-transform', 'rotate(' + degree + 'deg)');
		target_wp.css('-moz-transform-origin', '0% 50%');
		target_wp.css('-webkit-transform', 'rotate(' + degree + 'deg)');
		target_wp.css('-webkit-transform-origin', '0% 50%');
		target_wp.css('-o-transform', 'rotate(' + degree + 'deg)');
		target_wp.css('-o-transform-origin', '0% 50%');
		target_wp.css('-ms-transform', 'rotate(' + degree + 'deg)');
		target_wp.css('-ms-transform-origin', '0% 50%');
		prevDeg = degree
            }
        }
    }) // end mousemove
    $(document).mouseup(function (e) {
        dragging = false
	target_wp=""
    })
}
eventBroker = _({}).extend(require('chaplin/lib/event_broker'));
eventBroker.subscribeEvent('#doSave', function(state) {
    var states = {};
    var containDiv = $("#graphContainer").html();
    states = {"indexVal":indexVal,"graphContainer":containDiv ,"resetStatus":$('.icoReset').css('opacity')};
    var message = {
	type : 'state',
	data : JSON.stringify(states)
    };
    eventBroker.publishEvent("#save", message);
});
function saveFunction(){
    eventBroker.publishEvent("#doSave");
}
