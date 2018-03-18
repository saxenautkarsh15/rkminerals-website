
var pausescroller = (function(){

	var css3transformprop
	var css3transitionDuration
	var transitionendevt
	var cssanimationsupport
	
	function pausescroller(content, divId, delay, animationtime){
		this.initialized = false
		this.content //message array content
		this.tickerid=divId //ID of ticker div to display information
		this.delay=delay //Delay between msg change, in miliseconds.
		this.animationData = {currentVisibleDivTop: 0, currentHiddenDivTop: 0, dist: 0, animationTime: (animationtime || 100)}  // Transition related data
		this.mouseoverBol=0 //Boolean to indicate whether mouse is currently over scroller (and pause it if it is)
		this.starttimer = null // setTimeout reference
		this.hiddendivpointer=1 //index of message array for hidden div
		var scrollerinstance=this
		window.addEventListener("DOMContentLoaded", function(){scrollerinstance.initialize(content)}, false)
		window.addEventListener("load", function(){scrollerinstance.initialize(content)}, false)
	}
	
	// -------------------------------------------------------------------
	// initialize()- Initialize scroller method.
	// -Get div objects, set initial positions, start up down animation
	// -------------------------------------------------------------------
	
	pausescroller.prototype.initialize=function(content){
		if (this.initialized){
			return
		}
		this.initialized = true
		this.tickerdiv=document.getElementById(this.tickerid)
		this.tickerdiv.innerHTML = '<div class="innerDiv" id="' + this.tickerid + '1"></div><div class="innerDiv" style="visibility: hidden" id="' + this.tickerid+'2"></div>'
		this.visiblediv = document.getElementById(this.tickerid+"1")
		this.hiddendiv=document.getElementById(this.tickerid+"2")
		this.populate(content)
		this.visibledivtop=parseInt(getCSSpadding(this.tickerdiv))
		css3transformprop = getcss3prop('transform') // get supported JS version of "transform"
		css3transitionDuration = getcss3prop('transition-duration') // get supported JS version of "transition-duration"
		transitionendevt = gettransitionend() // get supported JS version of "transitionend"
		cssanimationsupport = css3transformprop && css3transitionDuration && transitionendevt
		this.visiblediv.style[css3transitionDuration] = this.hiddendiv.style[css3transitionDuration] = this.animationData.animationTime + 'ms' // set transition-duration dynamically
		this.getinline(this.visiblediv, this.hiddendiv)
		this.hiddendiv.style.visibility="visible"
		var scrollerinstance=this
		this.tickerdiv.addEventListener('mouseover', function(){scrollerinstance.mouseoverBol=1}, false)
		this.tickerdiv.addEventListener('mouseout', function(){scrollerinstance.mouseoverBol=0}, false)
		setTimeout(function(){
				scrollerinstance.animateup()
		}, this.delay)
		this.visiblediv.addEventListener(transitionendevt, function(e){
			if (e.propertyName.indexOf('transform') != -1){
				scrollerinstance.getinline(scrollerinstance.hiddendiv, scrollerinstance.visiblediv)
				scrollerinstance.swapdivs()
				scrollerinstance.starttimer = setTimeout(function(){
					scrollerinstance.setmessage()
				}, scrollerinstance.delay)
			}
		}, false)
	}
	
	
	// -------------------------------------------------------------------
	// animateup()- Move the two inner divs of the scroller up and in sync
	// -------------------------------------------------------------------
	
	pausescroller.prototype.animateup=function(){
		var scrollerinstance=this
		var animationData = this.animationData
		var visibledivtop = animationData.currentVisibleDivTop - animationData.dist
		var hiddendivtop = animationData.currentHiddenDivTop - animationData.dist
		if (cssanimationsupport){
			this.visiblediv.classList.remove('notransition')
			this.hiddendiv.classList.remove('notransition')
			this.visiblediv.style[css3transformprop] = 'translate3d(0, ' + visibledivtop + 'px, 0)'
			this.hiddendiv.style[css3transformprop] = 'translate3d(0, ' + hiddendivtop + 'px, 0)'	
		}
		else{ // legacy browsers
			this.visiblediv.style['top'] = visibledivtop + 'px'
			this.hiddendiv.style['top'] = hiddendivtop + 'px'
			this.getinline(this.hiddendiv, this.visiblediv)
			this.swapdivs()
			this.starttimer = setTimeout(function(){
				scrollerinstance.setmessage()
			}, this.delay)
		}
	}
	
	// -------------------------------------------------------------------
	// swapdivs()- Swap between which is the visible and which is the hidden div
	// -------------------------------------------------------------------
	
	pausescroller.prototype.swapdivs=function(){
		var tempcontainer=this.visiblediv
		this.visiblediv=this.hiddendiv
		this.hiddendiv=tempcontainer
	}
	
	pausescroller.prototype.getinline=function(div1, div2){
		var animationData = this.animationData
		var visibledivtop = this.visibledivtop
		var hiddendivtop = Math.max(div1.parentNode.offsetHeight, div1.offsetHeight)
		if (cssanimationsupport){
			div1.classList.add('notransition')
			div2.classList.add('notransition')
			div1.style[css3transformprop] = 'translate3d(0, ' + visibledivtop + 'px, 0)'
			div2.style[css3transformprop] = 'translate3d(0, ' + hiddendivtop + 'px, 0)'
		}
		else{ // legacy browsers
			div1.style['top'] = visibledivtop + 'px'
			div2.style['top'] = hiddendivtop + 'px'
		}
		animationData.currentVisibleDivTop = visibledivtop
		animationData.currentHiddenDivTop = hiddendivtop
		animationData.dist = hiddendivtop - visibledivtop
	}
	
	// -------------------------------------------------------------------
	// setmessage()- Populate the hidden div with the next message before it's visible
	// -------------------------------------------------------------------
	
	pausescroller.prototype.setmessage=function(){
		var scrollerinstance=this
		if (this.mouseoverBol==1) //if mouse is currently over scoller, do nothing (pause it)
			scrollerinstance.starttimer = setTimeout(function(){scrollerinstance.setmessage()}, 500)
		else{
			var i=this.hiddendivpointer
			var ceiling=this.content.length
			this.hiddendivpointer=(i+1>ceiling-1)? 0 : i+1
			this.hiddendiv.innerHTML=this.content[this.hiddendivpointer]
			scrollerinstance.animateup()
		}
	}

	// -------------------------------------------------------------------
	// PUBLIC: populate(), pause() and resume() functions
	// -------------------------------------------------------------------

	pausescroller.prototype.populate = function(contentarray){
		this.content = contentarray
		this.visiblediv.innerHTML = contentarray[0]
		this.hiddendiv.innerHTML = contentarray[1]
	}

	pausescroller.prototype.pause = function(){
		var scrollerinstance=this
		clearTimeout(scrollerinstance.starttimer)
	}

	pausescroller.prototype.resume = function(){
		var scrollerinstance=this
		clearTimeout(scrollerinstance.starttimer)
		scrollerinstance.starttimer = setTimeout(function(){scrollerinstance.setmessage()}, 500)
	}

	// -------------------------------------------------------------------
	// Helper functions
	// -------------------------------------------------------------------
	
	function getCSSpadding(tickerobj){ //get CSS padding value, if any
		if (tickerobj.currentStyle)
		return tickerobj.currentStyle["paddingTop"]
		else if (window.getComputedStyle) //if DOM2
		return window.getComputedStyle(tickerobj, "").getPropertyValue("padding-top")
		else
		return 0
	}
	
	function getcss3prop(cssprop){
	  var css3vendors = ['', '-moz-','-webkit-','-o-','-ms-','-khtml-']
	  var root = document.documentElement
	  function camelCase(str){
	    return str.replace(/\-([a-z])/gi, function (match, p1){ // p1 references submatch in parentheses
	      return p1.toUpperCase() // convert first letter after "-" to uppercase
	    })
	  }
	  for (var i=0; i<css3vendors.length; i++){
	    var css3propcamel = camelCase( css3vendors[i] + cssprop )
	    if (css3propcamel.substr(0,2) == 'Ms') // if property starts with 'Ms'
	      css3propcamel = 'm' + css3propcamel.substr(1) // Convert 'M' to lowercase
	    if (css3propcamel in root.style)
	      return css3propcamel
	  }
	  return undefined
	}
	
	function gettransitionend(){
	  var transitions = {
	    'transition':'transitionend',
	    'OTransition':'oTransitionEnd',
	    'MozTransition':'transitionend',
	    'WebkitTransition':'webkitTransitionEnd'
	  }
		var transitionprop = getcss3prop('transition')
		return transitions[transitionprop] || undefined
	}

	return pausescroller

})();


//CSS classlist pollyfill: https://github.com/remy/polyfills/blob/6e87470526c496c0fc53fa87ed5a825eff61f1f3/classList.js

(function () {

if (typeof window.Element === "undefined" || "classList" in document.documentElement) return;

var prototype = Array.prototype,
    push = prototype.push,
    splice = prototype.splice,
    join = prototype.join;

function DOMTokenList(el) {
  this.el = el;
  // The className needs to be trimmed and split on whitespace
  // to retrieve a list of classes.
  var classes = el.className.replace(/^\s+|\s+$/g,'').split(/\s+/);
  for (var i = 0; i < classes.length; i++) {
    push.call(this, classes[i]);
  }
};

DOMTokenList.prototype = {
  add: function(token) {
    if(this.contains(token)) return;
    push.call(this, token);
    this.el.className = this.toString();
  },
  contains: function(token) {
    return this.el.className.indexOf(token) != -1;
  },
  item: function(index) {
    return this[index] || null;
  },
  remove: function(token) {
    if (!this.contains(token)) return;
    for (var i = 0; i < this.length; i++) {
      if (this[i] == token) break;
    }
    splice.call(this, i, 1);
    this.el.className = this.toString();
  },
  toString: function() {
    return join.call(this, ' ');
  },
  toggle: function(token) {
    if (!this.contains(token)) {
      this.add(token);
    } else {
      this.remove(token);
    }

    return this.contains(token);
  }
};

window.DOMTokenList = DOMTokenList;

function defineElementGetter (obj, prop, getter) {
    if (Object.defineProperty) {
        Object.defineProperty(obj, prop,{
            get : getter
        });
    } else {
        obj.__defineGetter__(prop, getter);
    }
}

defineElementGetter(Element.prototype, 'classList', function () {
  return new DOMTokenList(this);
});

})();