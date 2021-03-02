## How to add multiple themes to your website with CSS and JS

I've been working on my portfolio website of late and wanted to add the "theme switch" where users get to select whether they want to view my website in dark mode or light mode. While working on that, I realized this can be extended to not just two, but as many themes as you'd like. Pretty cool, eh? Let's see how we can do that.

First, take a look at what it'll look like:

This is the light mode:
 
![light-mode.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1613684987839/FVXHpNEhN.png)

And this is the dark mode:


![dark-mode.PNG](https://cdn.hashnode.com/res/hashnode/image/upload/v1613685002620/PxeKZmHsJ.png)

And the theme switch is in the right top corner

First of all, let's define all your CSS in one file, and the CSS variables for the colors we want to change in another file. Let's call the one with all the CSS as our 
`style.css` and the ones with dark theme colors as `dark-variables.css` and likewise the one with light theme colors as `light-variables.css`

Let's take a look at the two files now:

light-variables.css:

```
:root {
	--background: antiquewhite;
	--font: #242526;
}

``` 

dark-variables.css
```
:root {
	--background: #242526;
	--font: antiquewhite;
}

```

That's great. Now, let's add them to our `index.html` like this. Note that the link for variables file has an id `stylesheet`. We will be using this later:

```
  <link id="stylesheet" rel="stylesheet" href="dark-variables.css" />
  <link rel="stylesheet" href="style.css" />

```

Here, I want the users to land on the dark-themed page by default and they can change it to the light theme if they want to. If you want it the other way, you can change the first stylesheet to `light-variables.css` like below:
```
<link id="stylesheet" rel="stylesheet" href="light-variables.css" />
<link rel="stylesheet" href="style.css" />
```
Next, we need to add an `event listener` to the theme switch button. Let's first look at what that button looks like:

```
 <div id="theme-switch" class="theme-switch">
          <div id="theme-icon" class="fas fa-moon"></div>
 </div>
```
It's basically a div with id `theme-switch` that contains another div with id `theme-icon` which basically uses  [font awesome](fontawesome.com)  icons. 

Now, let's add the event listener. What we need to do here is add a `click` event listener to trigger the theme switch function. 

Let's break that down into smaller chunks now:
- Let's get the theme-button first:

```
const themeButton = document.getElementById('theme-switch')
``` 

- Now, let's add the event listener to trigger the theme switch function:


```
themeButton.addEventListener('click', themeSwitch)
``` 

- Now, let's define the `themeSwitch` function bit by bit. First, let's get the stylesheet that is attached to the page currently:


```
const stylesheet = document.getElementById('stylesheet')
``` 

Remember we gave the id "stylesheet" to the variables file? That's what we are getting here.
- Next, let's check the href associated with the stylesheet. We can do that like:

```
const currentStyle = stylesheet.href
``` 

- Now, that we have the href, we can check which style is currently active and change to the other one. Let's do that:

```
   if (currentStyle.indexOf(lightTheme) !== -1) {
        stylesheet.href = darkTheme
        themeIcon.classList.remove(lightIcon)
        themeIcon.classList.add(darkIcon)
    }
    else {
        stylesheet.href = lightTheme
        themeIcon.classList.remove(darkIcon)
        themeIcon.classList.add(lightIcon)
    }
``` 


- If you've noticed that we are removing and adding another class there, you are correct. That's the icon itself which we want to change when the theme is changed. 
So, for the light theme, we want the icon to be a moon to indicate that the users can click that button to switch to the dark theme, and for the dark theme we want the icon to be a sun to indicate that they can switch to light theme.

- This is what the function looks like in the end. Notice the extra declarations there? I just prefer to assign variables to everything. That's my personal preference :) 

```
function themeSwitch () {
     const darkIcon = "fa-sun";
     const lightIcon = "fa-moon";
     const lightTheme = "light-variables.css";
     const darkTheme = "dark-variables.css";
    if (currentStyle.indexOf(lightTheme) !== -1) {
        stylesheet.href = darkTheme
        themeIcon.classList.remove(lightIcon)
        themeIcon.classList.add(darkIcon)
     
    }
    else {
        stylesheet.href = lightTheme
        themeIcon.classList.remove(darkIcon)
        themeIcon.classList.add(lightIcon)   
     
    }

}

```

There it is! We have a website that supports multiple themes! If you want to add more themes, you can just add more buttons and add an event listener to each one. Each of these buttons can have their stylesheet with the colors of your choice. 😎





If you liked this article and want to know more about the stuff I am building, let's stay in touch on  [Twitter ](https://twitter.com/TejanshRana) where I regularly post about things I am working on ❤