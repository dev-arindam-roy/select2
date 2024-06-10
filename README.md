# Select2
## Customized select2 dropdown application - dynamic rendering

### Check It!
[https://dev-arindam-roy.github.io/select2/](https://dev-arindam-roy.github.io/select2/)

```js
$(document).ready(function() {
    $('#select1').select2({
        width: '100%',
        allowClear: true,
        //placeholder: 'Select an option',
        dropdownPosition: 'below',
        templateResult: format,
        escapeMarkup: function(m) {
            return m;
        }
    });
    function format(data) {
        if (typeof data.id == "undefined" || !data.id) return data.text;
        var _optSize = $(data.element).data('size') ?? '';
        var _optColor = $(data.element).data('color') ?? '';
        var _optSku = $(data.element).data('sku') ?? ''; 
        var $container = $(`<div class="select2-result-repository clearfix">
                <div class="select2-result-repository__avatar"><img src="cubes_32.png" /></div>
                <div class="select2-result-repository__meta">
                    <div class="select2-result-repository__title">${data.text}</div>
                    <div class="select2-result-repository__description">SKU:${_optSku} | Size:${_optSize} | Color:${_optColor}</div>
                </div>
            </div>`
        );
        
        return $container;
    }
    $('#select1').on('select2:open', () => {
        $(".select2-results:not(:has(a))").prepend('<a href="javascript:void(0);" class="select2-add-new-option" data-select2id="select1" data-modal-id="addNewItemModal"><i class="fas fa-plus"></i> Add New Item</a>');
    });
    $('body').on('click', '.select2-add-new-option', function(e) {
        e.preventDefault();
        $('#' + $(this).data('select2id')).select2('close');
        $('#' + $(this).data('modal-id')).modal({
            backdrop: 'static', 
            keyboard: false
        }, 'show');
    });
    $('#addNewItemModal').on('shown.bs.modal', function () {
        $('div.onex-error').remove();
    });
    $("#frmx").validate({
        errorClass: 'onex-error',
        errorElement: 'div',
        ignore: '.ignore',
        rules: {
            item_name: {
                required: true,
                maxlength: 30
            },
            item_sku: {
                required: true,
                maxlength: 20
            },
            item_color: {
                required: true,
                maxlength: 60
            },
            item_size: {
                required: true,
                digits: true
            }
        },
        messages: {
            item_name: {
                required: 'Please enter item name',
                maxlength: 'Maximum 30 chars accepted'
            },
            item_sku: {
                required: 'Please enter item sku',
                maxlength: 'Maximum 20 chars accepted'
            },
            item_color: {
                required: 'Please enter item color',
                maxlength: 'Maximum 60 chars accepted'
            },
            item_size: {
                required: 'Please enter item size',
                digits: 'Please enter valid mobile number'
            }
        }
    });
    $('body').on('click', '#addItemBtn', function() {
        if ($('#frmx').valid()) {
            let data = {
                id: Math.random() * 1000,
                text: document.getElementById('itemName').value
            };
            var newOption = new Option(data.text, data.id, false, false);
            $('#select1').append(newOption).val(data.id).trigger('change');
            $('#select1').find(':selected')
                .attr('data-size', document.getElementById('itemSize').value)
                .attr('data-color', document.getElementById('itemColor').value)
                .attr('data-sku', document.getElementById('itemSku').value);
            $('#select1-error').hide();
            $('#frmx').find('.form-control').val('');
            $('#addNewItemModal').modal('hide');
        }
    });
    $('#getValueBtn').on('click', function() {
        //alert($('#select1').select2('val'));
        alert($('#select1').val());
    });
});
```

```js
$('.onex-select2').select2({
    width: '100%',
    allowClear: true,
    placeholder: 'Select an option',
    dropdownPosition: 'below'
});
```

```js
$('body').on('select2:select', '.onex-select2', function (e) { 
    if($(this).val() != '') {
        $('#' + $(this).attr('id') + '-error').hide();
        $(this).next('span.select2-container').removeClass('select2-custom-error');
        $(this).parent().find('.onex-form-lebel').removeClass('onex-error-label');
    }
});
```

```js
$('.onex-select2-ajax-user').select2({
    placeholder: 'Select Customer',
    allowClear: true,
    width: '100%',
    dropdownPosition: 'below',
    minimumInputLength: 2,
    maximumInputLength: 20,
    minimumResultsForSearch: 10,
    //selectOnClose: true,
    //closeOnSelect: false,
    //multiple: false,
    //debug: true,
    ajax: {
        url: "{{ route('ajax.getUsers') }}",
        dataType: "json",
        type: "GET",
        cache: true,
        delay: 250,
        quietMillis: 100,
        data: function (params) {
            var query = {
                search: params.term,
                page: params.page || 1,
                type: 'public',
                search_tag: 'customer',
                role_id: 6
            }
            return query;
        },
        processResults: function (data, params) {
            params.page = params.page || 1;
            return {
                results: $.map(data.restlt_data, function (item) {
                    return {
                        text: item.name,
                        id: item.user_id,
                        phno: item.phone_number,
                        loginid: item.login_id,
                        disabled: params.page == 2 ? true : false
                    }
                }),
                pagination: {
                    more: (params.page * 10) < data.data_count
                }
            };
        }
    },
    escapeMarkup: function (markup) { return markup; },
    templateResult: select2UserOptionFormat,
    templateSelection: createOrderSelect2TempSelection
});

function select2UserOptionFormat (data) {
    if (data.loading) {
        return loadingPlaceholder;
    }

    var $container = $(
        "<div class='select2-result-repository clearfix'>" +
            "<div class='select2-result-repository__avatar'><i class='far fa-user fa-2x'></i></div>" +
            "<div class='select2-result-repository__meta'>" +
                "<div class='select2-result-repository__title'></div>" +
                "<div class='select2-result-repository__description'></div>" +
            "</div>" +
        "</div>"
    );

    $container.find(".select2-result-repository__title").text(data.text);
    $container.find(".select2-result-repository__description").html(`<i class="fas fa-mobile-alt"></i> ${data.phno} | <i class="far fa-address-card"></i> ${data.loginid || ''}`);

    return $container;
}

function createOrderSelect2TempSelection (data) {
    return data.name || data.text;
}
```

## Links

[https://select2.org/getting-started/installation](https://select2.org/getting-started/installation)

[https://select2.org/programmatic-control/methods](https://select2.org/programmatic-control/methods)

[https://select2.org/programmatic-control/events](https://select2.org/programmatic-control/events)

[https://select2.org/programmatic-control/add-select-clear-items](https://select2.org/programmatic-control/add-select-clear-items)

[https://select2.org/configuration/data-attributes](https://select2.org/configuration/data-attributes)

[https://select2.org/upgrading/migrating-from-35#renamed-templating-options](https://select2.org/upgrading/migrating-from-35#renamed-templating-options)
