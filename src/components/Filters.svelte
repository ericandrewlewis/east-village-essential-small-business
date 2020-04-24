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
            color: '#99cfec',
            i18nIdentifier: 'filters.categories.groceries',
        },
        'health': {
            selectedColor: '#7cb342',
            color: '#cae0b3',
            i18nIdentifier: 'filters.categories.health',
        },

        'laundromat': {
            selectedColor: '#681a16',
            color: '#A47573',
            i18nIdentifier: 'filters.categories.laundromat',
        },
        'restaurants': {
            selectedColor: '#eb6565',
            color: '#f7c1c1',
            i18nIdentifier: 'filters.categories.restaurants',
        },
        'dessert': {
            selectedColor: '#9c27b0',
            color: '#d7a8df',
            i18nIdentifier: 'filters.categories.dessert',
        },
        'retail': {
            selectedColor: '#fff35f',
            color: '#fffabf',
            i18nIdentifier: 'filters.categories.retail',
        },
        'shops & services': {
            selectedColor: '#f2983f',
            color: '#f9d5b2',
            i18nIdentifier: 'filters.categories.shops_and_services',
        },
        'free food': {
            selectedColor: '#12ba96',
            color: '#8be3cc',
            i18nIdentifier: 'filters.categories.free_food',
        }
    }

    const subcategoryNameToI18nIdentifier = {
        'Takeout': 'filters.options.takeout',
        'Delivery': 'filters.options.delivery',
        'Shipping': 'filters.options.shipping',
    }

    let overallCategoryItems = []
    let subCategoryItems = []
    let textSearch = '' //todo - add textSearch filter

    $: {
        //init filters
        if ($data && $data.features.length > 0) {
            // Extract category names from DB
            const categoriesInDB = new Set($data.features.map(feature => feature.properties.overallcategory.trim().toLowerCase()))
            // Only display categories that are in the DB
            overallCategoryItems = [];
            // debugger;
            for (let customColor of Object.keys(customColors)) {
                if (categoriesInDB.has(customColor)) {
                    overallCategoryItems.push({
                        name: customColor,
                        selected: false,
                        i18nIdentifier: customColors[customColor].i18nIdentifier,
                    })
                }
            }
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
                console.log('what.')
                subCategoryItems = unique.filter(item => item.length > 0).map(item => ({
                    name: item,
                    selected: false,
                    i18nIdentifier: subcategoryNameToI18nIdentifier[item]
                }))
            } else {
                subCategoryItems = []
            }
        }
    }

    let optionItems = [
        {
            i18nIdentifier: 'filters.options.takeout',
            selected: false,
            filter: feature => feature.properties.takeoutyorn.toUpperCase() === 'Y'
        },
        {
            i18nIdentifier: 'filters.options.delivery',
            selected: false,
            filter: feature => feature.properties.deliveryyorn.toUpperCase() === 'Y'
        },
        {
            i18nIdentifier: 'filters.options.shipping',
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

<CategoryFilter name={$_('filters.categories.title')} categories={overallCategoryItems} unfilteredCategoryI18nIdentifier={'filters.categories.all'}
                on:update={e => overallCategoryItems = e.detail} {customColors}/>

<br>

<CategoryFilter name={$_('filters.subcategories')} categories={subCategoryItems} showAfter={1}
                on:update={e => subCategoryItems = e.detail}/>

<br>

<OptionFilter name={$_('filters.options.title')} options={optionItems} on:update={e => optionItems = e.detail}/>

<br>
