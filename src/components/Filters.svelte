<script>
    import {_} from 'svelte-i18n'
    import {data, filters} from '../stores'
    import GeneralSearch from './GeneralSearch.svelte'
    import {capitalizeFirstLetter} from '../utils/textFormating'
    import CategoryFilter from './filters/CategoryFilter.svelte'
    import OptionFilter from './filters/OptionFilter.svelte'

    const customColors = {
        'groceries': {
            selectedColor: '#0288d1',
            color: '#99cfec'
        },
        'health': {
            selectedColor: '#7cb342',
            color: '#cae0b3'
        },

        'laundromat': {
            selectedColor: '#681a16',
            color: '#A47573'
        },
        'restaurants': {
            selectedColor: '#eb6565',
            color: '#f7c1c1'
        },
        'dessert': {
            selectedColor: '#9c27b0',
            color: '#d7a8df'
        },
        'retail': {
            selectedColor: '#fff35f',
            color: '#fffabf'
        },
        'shops & services': {
            selectedColor: '#f2983f',
            color: '#f9d5b2'
        },
        'free food': {
            selectedColor: '#12ba96',
            color: '#8be3cc'
        }
    }

    let overallCategoryItems = []
    let subCategoryItems = []
    let textSearch = '' //todo - add textSearch filter

    $: {
        //init filters
        if ($data && $data.features.length > 0) {
            const categories = new Set($data.features.map(feature => feature.properties.overallcategory))
            //sort by customColors's order
            const order = Object.keys(customColors)
            const unique = Array.from(categories).map(item => [order.indexOf(item.toLowerCase().trim()), item])
                    .sort().map(arr => capitalizeFirstLetter(arr[1]))
            overallCategoryItems = unique.filter(item => item.length > 0).map(item => ({
                name: item,
                selected: false
            }))

        }
    }


    //update subCategories
    $: {
        if ($data && overallCategoryItems.length > 0) {
            const overallCategory = overallCategoryItems.find(item => item.selected);
            if (overallCategory) {
                //get subCategories for the selected overallCategory
                const subCategories = $data.features.filter(feature => capitalizeFirstLetter(feature.properties.overallcategory) === overallCategory.name)
                        .map(feature => capitalizeFirstLetter(feature.properties.subcategory))
                //filter for unique items
                const unique = Array.from(new Set(subCategories)).sort()
                subCategoryItems = unique.filter(item => item.length > 0).map(item => ({
                    name: item,
                    selected: false
                }))
            } else {
                subCategoryItems = []
            }
        }
    }

    let optionItems = [
        {
            name: $_('filters.options.takeout'),
            selected: false,
            filter: feature => feature.properties.takeoutyorn.toUpperCase() === 'Y'
        },
        {
            name: $_('filters.options.delivery'),
            selected: false,
            filter: feature => feature.properties.deliveryyorn.toUpperCase() === 'Y'
        },
        {
            name: $_('filters.options.shipping'),
            selected: false,
            filter: feature => feature.properties.shippingyorn.toUpperCase() === 'Y'
        },
    ]


    //function to generate filters
    $: {
        const overallCategoryFilter = (feature) => {
            const filterItems = overallCategoryItems.filter(item => item.selected);
            if (filterItems.length) {
                return filterItems.map(item => item.name).includes(capitalizeFirstLetter(feature.properties.overallcategory))
            }
            return true;

        }

        const subCategoryFilter = (feature) => {
            const filterItems = subCategoryItems.filter(item => item.selected);
            if (filterItems.length) {
                return filterItems.map(item => item.name).includes(capitalizeFirstLetter(feature.properties.subcategory))
            }
            return true;

        }

        const optionFilter = (feature) => {
            const filterItems = optionItems.filter(item => item.selected);
            if (filterItems.length > 0) {
                return filterItems.every(item => item.filter(feature))
            }
            return true;
        }

        filters.set([overallCategoryFilter, subCategoryFilter, optionFilter])
    }


</script>

<GeneralSearch {textSearch}/>

<CategoryFilter name={$_('filters.categories')} categories={overallCategoryItems}
                on:update={e => overallCategoryItems = e.detail} {customColors}/>

<br>

<CategoryFilter name={$_('filters.subcategories')} categories={subCategoryItems} showAfter={1}
                on:update={e => subCategoryItems = e.detail}/>

<br>

<OptionFilter name={$_('filters.options.title')} options={optionItems} on:update={e => optionItems = e.detail}/>

<br>
