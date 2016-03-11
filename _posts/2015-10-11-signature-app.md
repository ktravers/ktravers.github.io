---
layout: post
title: How to Add a Modular Signature App Using Rails, Backbone and Marionette
---

The view:
```slim
= form_tag enrollments_path, method: :post
  .signature-pad
    .form-row
      .canvas-text X Sign your name here
      canvas.signature-canvas width="600" height="200"
    .form-row
      .clear-btn.small Clear signature

  .buttons
    .submit-btn
      button type="submit" class="btn btn-default enrollment-submit" Submit Payment
    .cancel-btn
      = link_to 'Cancel', enrollments_landing_path, class: 'btn btn-default'
```


The js:
```javascript
var EnrollmentNewItemView = Marionette.ItemView.extend({
  ui: {
    'canvas': '.signature-canvas',
    'canvasText': '.canvas-text',
    'clearBtn': '.clear-btn',
    'cancelLink': '.cancel-btn a'
  },

  events: {
    'submit': 'onSubmit',
    'mousedown @ui.canvas': 'clearCanvasText',
    'mouseup @ui.canvas': 'toggleSubmit',
    'click @ui.clearBtn': 'onClearClicked'
  },

  initialize: function() {
    this.$button = this.$el.find('.enrollment-submit');
    this.disableSubmit();
  },

  onRender: function() {
    this.canvas = document.querySelector('canvas');
    this.signaturePad = new SignaturePad(this.canvas, {
      onBegin: function() {
        if (!this.signed) { this.clear(); }
        this.signed = true;
      }
    });
  },

  onSubmit: function(e) {
    e.preventDefault();

    this.disableSubmit();
    this.disableCancel();
    this.saveEnrollment();
  },

  onClearClicked: function() {
    this.signaturePad.clear();
    this.signaturePad.signed = false;
    this.disableSubmit();
    this.showCanvasText();
  },

  showCanvasText: function() {
    this.ui.canvasText.css('opacity', '1');
    this.ui.canvasText.css('z-index', '1');
  },

  clearCanvasText: function() {
    this.ui.canvasText.css('opacity', '0');
    this.ui.canvasText.css('z-index', '-1');
  },

  toggleSubmit: function(){
    if (!this.signaturePad.signed) {
      this.disableSubmit();
    } else {
      this.enableSubmit();
    }
  },

  enableSubmit: function() {
    this.$button.prop('disabled', false);
  },

  disableSubmit: function() {
    this.$button.prop('disabled', true);
  },

  enableCancel: function() {
    this.ui.cancelLink.attr('href', '/enrollments/welcome');
    this.ui.cancelLink.css({
      'cursor': 'pointer'
    });
  },

  disableCancel: function() {
    this.ui.cancelLink.removeAttr('href');
    this.ui.cancelLink.css({
      'cursor': 'default',
      'color': '#3B3B3B',
      'background-color': '#fff'
    });
  },

  saveEnrollment: function() {
    if ( this.signaturePad.signed ) {
      var signature = this.signaturePad.toDataURL();

      $.post(
        '/enrollments',
        { 'signature': signature }
      )
      .done(function(data) {
        window.location.href = data.redirect_path;
      })
      .fail(function(xhr, status, err) {
        // error handling
      }.bind(this));

    } else {
      this.toggleSubmit();
      this.enableCancel();
      // error handling
    }
  }
});

module.exports = EnrollmentNewItemView;
```
