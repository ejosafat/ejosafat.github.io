---
layout: post
title:  "Write expressive code (I)"
date:   2013-08-05
categories: 2013 
tags: other 
permalink: write-expressive-code-I 
published: true
---
# Write expressive code (I)
When you work in a large project with many programmers, it's frequent to be assigned a task involving code written by another guy. It's pretty nice to find a clean code understandable at first glance, where the intention of each line is explained by itself but…

…that's a beautiful dream which seldom comes true, so after a painful time reading and trying to figure out what the hell is going on, why don't refactor to clarify it? Doing this before coding any new feature avoids producing more garbage.

> *Never express yourself more clearly than you are able to think.*
> *([Niels Bohr, Danish physicist](http://en.wikipedia.org/wiki/Niels_Bohr))*

> *If you don't express yourself clearly, it could be interpreted as you are unable to think.*
> *(Unknown Jedi Master)*

## A real case
Once upon a time I was assigned to add a new validation check on  some fields of a file upload dialog which uses [JQuery File Upload Plugin](https://github.com/blueimp/jQuery-File-Upload). The point of insertion of the new check should be the *add* callback. I found a "clear" example of people adding more and more functionality inside the callback without simplifying it previously (or at least formatting it!!!):

```javascript
add : function (e, data) {
  file_ext = data.files[0].name.split('.').pop().toLowerCase()
  format = gon.formatFormats[file_ext]
  $('#upload-file').removeClass('error')
  $('#upload-file').children('.controls').children("ul.help-block").remove()
        if ((format == 'video') || (format == 'audio') || (format == undefined)){
    $("select#format_id").children().first().attr("selected","selected")
          if ((format == 'video') || (format == 'audio')){
      $('#upload-file').addClass('error')
      $('#upload-file').children('.controls').append("<ul class='help-block'><li>Content file not allowed</li></ul>")
          } else {
          $('input[type=submit]').removeClass('disabled')
          }
        } else {
        $('input[type=submit]').removeClass('disabled')
      $("select#format_id").children().each(function(){
      if($(this).text() == format) { 
        $(this).attr("selected","selected")
  $("select#format_id").parent('.controls').parent().removeClass('error')
  $("select#format_id").parent('.controls').children("ul.help-block").remove()
      }
    });
        }
 
        that.model.setFile(data.files[0])
  that.uploadFile = data

        return false
      },
```

## Clarify it
Unformatted code, deeply nested conditionals… after blaming myself for the decision to spend my life working as a programmer, I focused on extracting methods out of the code inside the conditionals to clarify what was going on there. The result:

```javascript
add : function (e, data) {
  file_ext = data.files[0].name.split('.').pop().toLowerCase()
  format = gon.formatFormats[file_ext]
  $('#upload-file').removeClass('error')
  $('#upload-file').children('.controls').children("ul.help-block").remove()

  if (that._isForbiddenFormat(format) || (format == undefined)) {
    that._selectFirstOptionInFormatDropdown()
    if (that._isForbiddenFormat(format)) {
      that._showFileFormatError()
      that._disableSubmit()
    } else {
      that._enableSubmit()
    }
  } else {
    that._enableSubmit()
    that._setFormatDropdown(format)
  }

  that.model.setFile(data.files[0])
  that.uploadFile = data

  return false
},
```

## Further refactoring to add the new check
After reading the code with those well named function calls, I could figure out the execution flow and refactor it without those pesky nested conditionals, and adding the new feature (a call to *that._requiredFieldsPresent()*) was a matter of minutes. What a precious time I'd have saved if I had found a code like this:
 
```javascript
add : function (e, data) {
  var format

  that._removeFileErrors()

  format = that._getFileFormat(data)
  if (that._isForbiddenFormat(format)) {
    that._showFileFormatError()
    that._disableSubmit()
    that.uploadFile = null
    return
   }
   that._setFormatDropdown(format)

   that.model.setFile(data.files[0])
   that.uploadFile = data

   if (that._requiredFieldsPresent()) {
    that._enableSubmit()
   }

   return false
},
```

## To keep in mind
* After coding it, take a moment to think about what you've written. Most of the times there is a chance to extract well named methods (it will take a few minutes) to express your intention.  
* **Expressive code is more meaningful and useful than outdated comments**.
