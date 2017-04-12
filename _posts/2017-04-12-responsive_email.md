---
layout: post
title: "Responsive e-mails: tips and tricks"
description: 
tags: [email, html, css, mailing]
categories: [front-end]
---

Creating a responsive e-mail may give you strong headaches.  
There are so many things to deal with!
It is even more frustrating and time-consuming when you are stuck with a problem you are not able to solve, and finding the root cause can be a pain:

- You have to deal with "bad practices"
- Each e-mail client has its own CSS support
- Some e-mail clients apply their own style
- In some cases, e-mail clients modify the DOM of your e-mail
- And many other atrocities that will make you have nightmares

This blog article aims to list the issues we encountered when creating our own responsive e-mails and give you leads to resolve similar issues.  
I don't claim to have THE solution, nor to be exhaustive. But I hope it will help.

## The margin property doesn't seem to work in Outlook.com

You might have noticed that your margins are ignored under Outlook.com. And you are right!  
Outlook.com dropped the margin support as you can see in [https://www.campaignmonitor.com/css/](https://www.campaignmonitor.com/css/).  
That is also the case for Office 365 (which is an Outlook.com you have to pay for).

Hopefully, there is a trick : **Capitalize your margin property**

Instead of writing `margin: 10px;`  
**Use `Margin: 10px;`**

## When I apply margin or padding on a table, Outlook 2007 and 2010 acts weird

When applying CSS margin or padding to a `table`, Outlook will apply the same margin or padding to every nested `td`.  
Using these CSS properties on the containing `table element` should be avoided  

The `cellpadding` and `cellspacing` properties are safe though.

## Positioning with floats and positions does not work

As half of the most used e-mail clients dropped the CSS support of `float` and `position`, these two properties are to avoid in your developments.  
Use HTML tables as workaround.

What is known as a bad practice in HTML development becomes a good one when it comes to e-mails.

## Text and some elements look bigger in some e-mail client

Some e-mail clients apply their own style and can make your e-mails look a little bit different than what you expect  
To avoid those unwanted behaviors, you should size every element with `font-size`, `width` or `max-width`.

## Max-width does not work under Outlook

When you deal with responsive e-mails, you certainly use the `max-width` property to limit the width of fluid elements on larger screens.  
And this is definitely a good practice... Which doesn't work in Outlook, as it has dropped the support for this CSS property.  
It means that, in Outlook, an element with a `max-width` property will take **all the available place**.

The trick here is to wrap all your fluid elements into a container with fix size, in a HTML conditional comment that targets Outlook.

{% highlight html %}
<!--[if (gte mso 9)|(IE)]>
    <table width="430" align="center" border="0" cellpadding="0" cellspacing="0">
        <tr>
            <td>
<![endif]-->
<table align="center" border="0" cellpadding="0" cellspacing="0" style="max-width: 430px;">
    <tr>
        <td>
            [Content goes here]
        </td>
    </tr>
</table>
<!--[if (gte mso 9)|(IE)]>
            </td>
        </tr>
    </table>
<![endif]-->
{% endhighlight %}

This conditional expression targets all Microsoft Outlook (mso) versions greater than or equal to version 9 as well as Outlook versions that use Internet Explorer to render (like Outlook 2000–2003) and Lotus Notes.

## Table are not centered in Gmail (Safari)

If you try to center a table with the `align="center"` attribute, you may have noticed that the display is not the expected one in Gmail when you open it from Safari.  
For some obscure reasons, it just ignores it and the table is not centered.

The trick is to simply center the table with the CSS `Margin` property.

## Responsiveness is broken in Gmail in discussion mode

You may have noticed that Gmail detects the duplicate contents between the mail of a same conversation.  
It highlights these contents by coloring them.

To apply this new style, **Gmail changes the DOM of your e-mail by wrapping the duplicate contents into a div with a specific style**.  
This wrapper is obviously **not responsive** and breaks the responsiveness of your e-mail.

As the grouping by conversation depends on the recipient configuration, there's nothing you can do to avoid this unwanted behavior.  
The solution is to be careful when making your campaign.  
If you send **several similar e-mails** to **the same recipient**, then you must **change the subject of your e-mail**.

## There're unwanted gaps around images in table cells

Randomly, you can see a gap above and/or below your images in `<td>`. It's often a discrete gap of 1px that shows depending on the e-mail client or the screen resolution.  
Most of the time, you won't even notice it unless:
 - You have a background color under your image that is different from the other parts of the e-mail
 - You want to put images side by side in two differents table cells

### How to remove the gaps

To remove those gaps, you have to apply this style to your image:

{% highlight css %}
.img {
    display: block; /* Causes a line break before and after your image */
    border: none; /* Lotus Notes sometimes add borders around images */
    width: [feel free to choose the good one];
    height: auto; /* In rare occasions you'll still have a small gap. Use height: 100% in this case. Caution: it can change the image's ratio */
}
{% endhighlight %}

The `display: block;` trick is the most popular one when it comes to images.
But if it is not enough, you should take a look at [this page](https://www.emailonacid.com/blog/article/email-development/12_fixes_for_the_image_spacing_in_html_emails).

### The bottom gap in Outlook.com

Even with the previous trick, you won't prevent the gap in Outlook.com.  
The root cause is because **Oulook.com (and Office 365) wraps images in a HTML button**.
This causes a huge gap of 5px at the bottom of every image of your e-mail, and there's nothing you can do to prevent this behavior.

The only workaround is to wrap **all your images** in a `div` with the wanted height.
Fortunately, fixing the height will not break the responsiveness!