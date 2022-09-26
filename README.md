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

Then, add this javascript inside script tags in the product-template.liquid file :

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

Next, in the case of the Dawn theme, you need to update the theme.js file in the Assets folder. Find the code starting with _onSelectChange(). Inside the _onSelectChange() , add another method call for updateMeta(): after this.currentVariant = variant;

```
_onSelectChange: function() {
      var variant = this._getVariantFromOptions();
      this.$container.trigger({
        type: 'variantChange',
        variant: variant
      });
      if (!variant) {
        return;
      }
      this._updateMasterSelect(variant);
      this._updateImages(variant);
      this._updatePrice(variant);
      this._updateSKU(variant);
      this.currentVariant = variant;
      this.updateMeta();
      if (this.enableHistoryState) {
        this._updateHistoryState(variant);
      }
    },
```

Then add the updateMeta code further down with the other update methods:

```
/**
     * Trigger event when variant image changes
     *
     * @param  {object} variant - Currently selected variant
     * @return {event}  variantImageChange
     */
    _updateImages: function(variant) {
      var variantImage = variant.featured_image || {};
      var currentVariantImage = this.currentVariant.featured_image || {};

      if (
        !variant.featured_image ||
        variantImage.src === currentVariantImage.src
      ) {
        return;
      }

      this.$container.trigger({
        type: 'variantImageChange',
        variant: variant
      });
    },
/**
 * update metafield value when change the option
 */
    updateMeta() {
      extraVariantInfo(this.currentVariant.id);
    },
    /**
     * Trigger event when variant price changes.
     *
     * @param  {object} variant - Currently selected variant
     * @return {event} variantPriceChange
     */
    _updatePrice: function(variant) {
      if (
        variant.price === this.currentVariant.price &&
        variant.compare_at_price === this.currentVariant.compare_at_price
      ) {
        return;
      }
      this.$container.trigger({
        type: 'variantPriceChange',
        variant: variant
      });
    },
```

Now your variant EM Variant description should be appearing and updating as you select different variant options.

