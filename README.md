# Adding Product Variants Description into the Dawn Theme

First, here's how to display description depending on a selected product variant :   

Add html to the main-product.liquid file in the Sections folder (position it where you would like to display the description), for example: 

```
<div class="hideAll">
   <p>
    <span> Variant Description: </span>
    <span class="em-secondary-desc"></span>
   </p>
</div>
```
Below this add some liquid code to capture the associated metafield data and store it in a variable, here called em_secondary_desc_meta :

```
{% capture 'em_secondary_desc' %}
  {% for variant in product.variants %}
    {{variant.id}}:{{ variant.metafields.em_secondary_desc.desc_value.value | json }}
      {% unless forloop.last %},{% endunless %}
  {% endfor %}
{% endcapture %}
```

Then, add this javascript inside script tags in the main-product.liquid file :

```
<script>
  const currentVariantId = {{ product.selected_or_first_available_variant.id }};
  const emSecondaryDesc = { {{ em_secondary_desc }} };
  const emVariantDescriptionInfo = (id) => {
  let selector = document.querySelector('.em-secondary-desc');
  let hide = document.querySelector('.hideAll')
    if (emSecondaryDesc[id]) {
     hide.style.display = 'block'
     selector.innerHTML = emSecondaryDesc[id];
    }
    else
     hide.style.display = 'none'
  }
  emVariantDescriptionInfo(currentVariantId);
</script>

```

Next, in the case of the Dawn theme, you need to update the theme.js file in the Assets folder. Find the code starting with onVariantChange(). Below that, inside the else block, add another method call for updateMeta():

```
onVariantChange() {
    this.updateOptions();
    this.updateMasterId();
    this.toggleAddButton(true, '', false);
    this.updatePickupAvailability();
    this.removeErrorMessage();

    if (!this.currentVariant) {
      this.toggleAddButton(true, '', true);
      this.setUnavailable();
    } else {
      this.updateMedia();
      this.updateURL();
      this.updateVariantInput();
      this.renderProductInfo();
      this.updateShareUrl();
      this.updateMeta();
    }
  }

```

Then add the updateMeta code further down with the other update methods:

```
updateOptions() {
    this.options = Array.from(this.querySelectorAll('select'), (select) => select.value);
  }

  updateMasterId() {
    this.currentVariant = this.getVariantData().find((variant) => {
      return !variant.options.map((option, index) => {
        return this.options[index] === option;
      }).includes(false);
    });
  }

  updateMeta() {
	  emVariantDescriptionInfo(this.currentVariant.id);
  }

  updateMedia() { ...
```

Now your variant EM Variant description should be appearing and updating as you select different variant options.

