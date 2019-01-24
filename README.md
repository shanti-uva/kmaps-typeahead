# Kmaps Typeahead

This project is based on [Shanti Kmaps Typeahead](https://github.com/shanti-uva/shanti_kmaps_typeahead), the idea behind this fork is to provide a simplified version that can be expanded incrementally and can be used through the whole Kmaps project.

The typeahead functionality is provided by a jQuery plug-in that extends the [Typeahead plug-in](https://github.com/corejavascript/typeahead.js).

## Getting Started

To add this plug-in to your Kmaps application you can download the code from the git repository:
* [https://github.com/shanti-uva/kmaps-typeahead](https://github.com/shanti-uva/kmaps-typeahead)

You can clone it or just download the jQuery plug-in inside the `js` folder.

### Prerequisites

The plug-in requires:

* Kmaps app (this plug-in was designed for the KMAPS apps so you need an instance for it to run)
* Solr index where the data is stored (and consulted)
* [Typeahead plug-in](https://github.com/corejavascript/typeahead.js)

### How to use it:

```
$(document).ready(function(){
        $("#tree").kmapsSimpleTypeahead({
          solr_index: 'HTTPS://SERVER/PATH/TO/TERM_INDEX',
          domain: 'APP DOMAIN',         //[places, subjects, terms]
          autocomplete_field: 'name_autocomplete', //the name of the main field to search in
          search_fields: ['name_tibt'], //Additional field names that can be searched with an OR
          max_terms: 150,               //Max number of terms returned
          min_chars: 1,                 //After how many characters start doing a search
          sort: '',                     //sort parameter to Solr query, e.g. header+ASC
          fields: '',                   //list of aditional fields to get from solar
          menu: '', //Menu settings
          no_results_msg: '',           //Message to display when no results are found
          match_criterion: 'contains',  //{contains, begins, exactly}
          case_sensitive: false,
          ignore_tree: false,           //This is useful when we want to search in all trees
          templates: {}                 //templates to overwrite the default templates
        };
});
```

### Additional search fields

When we would like to search additional fields we add them into the `search_fields` array, they will be joined using the OR operator. For example, in the case of Tibetan texts we should add the field `name_tibt` that is set up in Solr to handle the special Unicode cases.

### Menu settings

Menu represents the popover that displays all the typeahead options. The default values for menu are:

```
        {
          minLength: settings.min_chars, // minimum number of characters to wait before calling the query
          hint: true,                    //show a hint behind the inputbox
          classNames: {                  //Definis the names of the classes the plugin will add to the elements
            input: 'kmaps-tt-input',
            hint: 'kmaps-tt-hint',
            menu: 'kmaps-tt-menu',
            dataset: 'kmaps-tt-dataset',
            suggestion: 'kmaps-tt-suggestion',
            selectable: 'kmaps-tt-selectable',
            empty: 'kmaps-tt-empty',
            open: 'kmaps-tt-open',
            cursor: 'kmaps-tt-cursor',
            highlight: 'kmaps-tt-highlight'
          }
        }
```

### Templates

The templates object will be merged into the default templates object overwriting any default configuration. The default object contains the following elements:

```
      var templates = {
        pending: function () {
          // Gets called when the seach is being processed
          // Returns a String represenging the code to display while processing the search.
          return '<div class="kmaps-tt-message kmaps-tt-searching">Searching ...</div>';
        },
        header: function (data) {
          // Gets called to display the header of the result menu
          // The data value contains the query(the string searched) and the suggestions array that match that query.
          // Returns a String representing the header
          var results;
          if (data.query) {
            results = 'Results for "<span class="kmaps-tt-query">' + data.query + '</span>"';
          }
          else {
            results = 'All results';
          }
          var pager = !result_paging ? '' : KMapsUtil.getTypeaheadPager(Number(plugin.settings.max_terms), data.suggestions[0].index, data.suggestions[0].numFound);
          var header = '<div class="kmaps-tt-header kmaps-tt-results"><button class="close" aria-hidden="true" type="button">×</button>' + results + pager + '</div>';
          if (settings.domain == 'places') { // add column headers
            header += '<div class="kmaps-place-results-header"><span class="kmaps-place-name">Name</span> <span class="kmaps-feature-type">Feature Type</span></div>';
          }
          return header;
        },
        footer: function (data) {
          // Gets called to display the footer of the result menu
          // The data value contains the query(the string searched) and the suggestions array that match that query.
          // Returns a String representing the footer
          var pager = !result_paging ? '' : KMapsUtil.getTypeaheadPager(Number(plugin.settings.max_terms), data.suggestions[0].index, data.suggestions[0].numFound);
          return '<div class="kmaps-tt-footer kmaps-tt-results">' + pager + '</div>';
        },
        notFound: function (data) {
          // Gets called when there was no matches to the query
          // The data value contains the query(the string searched) and the suggestions array that match that query.
          // Returns a String 
          var msg = 'No results for <span class="kmaps-tt-query">' + data.query + '</span>. ' + settings.no_results_msg;
          return '<div class="kmaps-tt-message kmaps-tt-no-results">' + msg + '</div>';
        },
        suggestion: function (data) {
          // Gets called for each suggestion that matched the query
          // The data value contains the query(the string searched) and the fields object requested by the query on the object doc
          // Returns a String 
          var cl = [];
          if (data.selected) cl.push('kmaps-tt-selected');
          var display_path = data.doc.ancestors ? data.doc.ancestors.join("/") : "";
          if (settings.domain == 'places') { // show feature types
            cl.push('kmaps-place-result');
            var feature_types = data.doc.feature_types ? data.doc.feature_types.join('/') : '';
            return '<div data-id="' + settings.domain+"-"+data.id + '" data-path="' + display_path + '" class="' + cl.join(' ') + '"><span class="kmaps-place-name">' + data.value + '</span> <span class="kmaps-feature-type">' + feature_types + '</span>' + '</div>';
          } else { // show hierarchy
            return '<div data-id="' + settings.domain+"-"+data.id + '" data-path="' + display_path + '" class="' + cl.join(' ') + '"><span class="kmaps-term">' + data.value + '</span>' + '</div>';
          }
        }
      };

```

### How to handle the selection of a suggestion

To handle a selection we modify the default behaviour, the default behaviour is write the value of the suggestion to the input text. But if we would like to store some of the additional fields on a different input field we could do it like this:

```
  $("#simpleTestText").kmapsSimpleTypeahead({
    solr_index: $('#test_simple_typeahead_js_data').data('solrIndex'),
    domain: $('#test_simple_typeahead_js_data').data('domain'),
    match_criterion: 'contains', //{contains, begins, exactly}
    ignore_tree: true,
    fields: 'ancestor_id_tib.alpha_path', //Ask for the additional field ancestor_id_tib.alpha_path
    templates: template,
  }).bind('typeahead:select',
        function (ev, sel) {
        // Assuming we have an additional input field with id: simpleTestResult
        // set its value to the value of the requested field `ancestor_id_tib.alpha_path`
        $("#simpleTestResult").val(sel.doc.ancestor_id_path);
        }
      );

```


## Authors

* *Initial work* - [Yuji](https://github.com/ys2n)
* *Initial work* - [Ed](https://github.com/torma)
* Modifications - [Derik](https://github.com/rderik)


## Acknowledgements

* To everyone at [SHANTI](https://github.com/orgs/shanti-uva/people)
