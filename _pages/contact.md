---
layout: page
title: Contact
permalink: /contact/
description: I would be glad to help you. Please leave a note or just say hello.
---

### Write to me
I would be glad to help you. Please leave a note or just say hello.


<form class="needs-validation" action="{{site.data.main.smart-forms}}" method="POST" novalidate>
  <div class="form-group">
    <label for="email">Email address</label>
    <input type="email" name="email" class="form-control" placeholder="name@example.com" required>
    <div class="valid-feedback">
        Looks good!
    </div>
    <div class="invalid-feedback">
        Please provide a valid email.
    </div>
  </div>
  <div class="form-group">
    <label for="message">Your message</label>
    <textarea class="form-control" name="content" id="" rows="3" placeholder="Enter your message" required></textarea>
    <div class="valid-feedback">
        Looks good!
    </div>
    <div class="invalid-feedback">
        Please provide a message.
    </div>
  </div>
  <!-- <div class="g-recaptcha" data-sitekey="6LcI_nUaAAAAAOz_n3PiU8ynA1SxL2idPE6gkNqF"></div> -->
  <!-- <input type="hidden" name="_next" value="{{site.url}}{{page.url}}">
  <input type="hidden" name="_subject" value="New Contact Form Submission">
  <input type="text" name="_gotcha" style="display:none"> -->
  <!-- <div id='recaptcha' class="g-recaptcha"
         data-sitekey="6LcI_nUaAAAAAOz_n3PiU8ynA1SxL2idPE6gkNqF" data-size="invisible" data-callback='onSubmit'></div> -->
  <button type="submit" class="btn btn-success">Submit</button>
</form>

<!-- <script src="https://www.google.com/recaptcha/api.js"></script> -->

<!-- <script>
   function onSubmit(token) {
     document.getElementById("contact-form").submit();
   }
</script> -->

<script>
(function() {
  'use strict';
  window.addEventListener('load', function() {
    
    var forms = document.getElementsByClassName('needs-validation');

    var validation = Array.prototype.filter.call(forms, function(form) {
      form.addEventListener('submit', function(event) {
        if (form.checkValidity() === false) {
          event.preventDefault();
          event.stopPropagation();
        }
        form.classList.add('was-validated');
      }, false);
    });
  }, false);

})();
</script>
