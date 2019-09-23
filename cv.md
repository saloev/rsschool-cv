# Saloev Sadi

## Contacts
-----

_Address_ : __Torzhkovskaya Ulitsa, 15,
Saint-Petersburg__

_Email_ : __saloev.saadi@yandex.ru__

_Phone_ : __+79602337850__

_Social_ :
1. __[GitHub](https://github.com/saloev)__ 
2. __[Vk](https://vk.com/s.saloev)__

-----

## Summary
----
_I started to get interest in web development in early school years, gradually web development attracted me more and more. Eventually i decided to learn it more professionally and started my bachelor in web development. After in depth learning of web development I started to like frontend development. Working with frontend development gives me pleasure- the best work is the work you like._

_Now I’m learning more about computer science, algorithms and data structures and of course more about my favorite programing language JavaScript_
        

----

## Skills
----
### Main skills 
- HTML 
- CSS(SASS)  
- JavaScript 
- TypeSctipt 
- Vue(Vuex, Vue Router)
- Git  
- Bash   

### Soft skills
- Flexibility
- Integrity
- Teamwork
- Responsibility
- Positive Attitude
----

## Code examples
----

My latest code is a main filter component with Vue and TypeScript.
I just put a Typescript code and not put html and css code. 

This component send to API data of filter and changing route to "car stock" page. 
```vuejs
    import {Vue, Component, Prop, Watch} from 'vue-property-decorator';
    import {Getter, Action, Mutation} from "vuex-class"
    import PulseLoader from 'vue-spinner/src/PulseLoader.vue';

    import filterSlider from "./filterSlider.vue"
    import carsFilter from "./carsFilter.vue"
    import '@icons/arrow';

    @Component({
        components: {
            filterSlider,
            carsFilter,
            PulseLoader
        }
    })
    export default class MainFilter extends Vue {
        @Getter('stock') stock: any;
        @Getter('common') common: any;
        @Action('fetchCarList') fetchCarList: any;
        @Mutation('setDataLoadingStatus') setDataLoadingStatus: any;
        @Getter('isDataLoadingFromAPI') dataLoading: any;

        isOpen: boolean = true;
        isURLChanged = false;

        filter: any = {
            brandName: null,
            modelName: null,
            maxEngine: null,
            minEngine: null,
            maxYear: null,
            minYear: null,
            bodyId: null,
            fuelsId: null,
            transmission: null,
            drives: null,
            priceMax: null,
            priceMin: null,
            run: null,
            runMin: null,
            sourceId: null,

        };

        currentCars: number = 0;
        limitCars: number = 3;

        beforeMount() {
            this.applyURLToFilter();
        }

        mounted() {
            this.limitCars = this.stock.howManyLoad;
        }

        get howManyLoad() {
            return this.stock.howManyLoad;
        }

        get carsCount() {
            if (!this.stock.filter['carsCount']) return [];

            return this.stock.filter['carsCount'];
        }

        get allCarsCount() {
            return this.stock.countAllStock;
        }

        // марка
        get brandNames() {
            return this.fetchCommonFilters('tradeInCarBrands', 'name', undefined, 'id');
        }

        // модель
        get modelName() {
            if (!this.filter.brandName) return [];

            // reset modelName to default if brandNames change
            this.filter.modelName = null;

            const brandId = this.brandNames.filter(({value}: any) => value === this.filter.brandName)[0].id;
            return this.fetchCommonFilters('models', 'name', undefined, 'producerId').filter(({id}: any) => {
                return id === brandId;
            });
        }

        // кузов
        get bodies() {
            return this.fetchCommonFilters('bodies', 'name', 'id');
        }

        // коробка передач
        get transmissions() {
            return this.fetchCommonFilters('transmissions', 'transmission').filter((item: any) => item.value);
        }

        // привод
        get drives() {
            return this.fetchCommonFilters('drives', 'drive').filter((item: any) => item.value);
        }

        // цена
        get prices() {
            return this.fetchCommonFilterNumbers('filterPrices', 'minPrice', 'maxPrice');
        }

        // пробег
        get runs() {
            return this.fetchCommonFilterNumbers('filterRuns', 'minRun', 'maxRun');
        }

        // двигатель
        get fuels() {
            //@ts-ignore params has a default value !!
            return this.fetchCommonFilters('fuels', 'name', 'id');
        }

        // локации
        get locations() {
            if (!this.common || !this.common.relatedSources) return  [];

            return this.common.relatedSources.reduce((acc: any, item: any) => {
                const {
                    id,
                    name
                } = item;

                return [...acc, {text: name, value: id}];
            }, [])
        }


        get isFilterEmpty() {
            return Object.keys(this.filter).every((key: string) => !this.filter[key]);
        }

        get currentURL() {
            return this.$route.query;
        }

        onSubmitPriceMax(priceMax: any) {
        	this.resetFilter();
        	this.filter.priceMax = priceMax;

	        this.setDataLoadingStatus(true);
	        setTimeout(() => {
		        this.setDataLoadingStatus(false);
	        }, 2500);

        	this.$router.push({ name: 'InStock', query: { 'priceMax' : priceMax }});
        }

        fetchCommonFilters(name: string, keyName: string, id = keyName, needId: false) {
            const filters: any = this.stock.filter[name];
            if (!filters) return [];

            if (!needId) return filters.reduce((acc: any, item: any) => {
                return [...acc, ...[{
                    value: item[id],
                    text: item[keyName],
                }]]
            }, []);

            const res: any = filters.reduce((acc: any, item: any) => {
                return [...acc, ...[{
                    value: item[id],
                    text: item[keyName],
                    id: item[needId],
                }]]
            }, []);

            return res;
        }

        fetchCommonFilterNumbers(name: string, min: string, max: string) {
            const filter: any = this.stock.filter[name];
            if (!filter) return [];

            const res: any = {
                min: filter[min],
                max: filter[max],
            };

            return res;
        }

        applyURLToFilter() {
            const queryKeys = Object.keys(this.$route.query);
            queryKeys.map((key: string) => {
                this.filter[key] = this.$route.query[key];
                return;
            });

            return;
        }

        getCarList() {
            this.fetchCarList({params: {limit: ` limit ${this.currentCars}, ${this.limitCars}`}});
        }


        resetFilter() {
            return Object.keys(this.filter).map((key: string) => {
                this.filter[key] = null
            });
        }

        sendFilter() {
            const queries = Object.keys(this.filter)
                .filter((key: string) => this.filter[key])
                .reduce((acc: any, key: string) => {
                    return {...acc, ...{[key]: this.filter[key]}}
                }, {});
            const limit = ` limit ${this.currentCars}, ${this.limitCars}`;

            this.setDataLoadingStatus(true);

            this.$router.push({path: '/stock', query:
                    {
                        ...queries,
                        limit2: `${0}`,
                        limit1: `${this.howManyLoad}`,
                        'page': 1
                    }});

            setTimeout(() => {
                this.setDataLoadingStatus(false);
            }, 2500);
        }

    }
```
----

## Experience
----
### Self Experience
I most coding in JS. I use Vue js for web application and also I using TypeScript.

I love solving algorithmic tasks and that's why i'm coding at:
1. [Codewars](https://www.codewars.com/users/saloevv)
2. [Hexlet](https://ru.hexlet.io/u/gwynbleidd/courses)
3. [FreeCodeCamp](https://www.freecodecamp.org/saloev)

And of course i making UI with HTML and CSS. 

More in [GitHub](https://github.com/saloev)

### Work Experience

_10-12-2018 - until now_: 

____
Junior Web Developer

[Playnext, Saint-Petersburg](https://playnext.ru/)

Website development for all levels of business.
 
Website maintenance and promotion.

#### What I do at work ?
Frontend(more) - Vue, TypeSctipt, JS, HTML, CSS, Bootstrap, Element UI

Backend(less) - PHP, Mysql, Bitrix CMS. 

____

----

## Education
----

### Certification
_16-04-2018_ : [Javascript для начинающих](https://stepik.org/cert/102493)

_24-11-2018_: [Javascript Algorithms And Data Structures](https://www.freecodecamp.org/certification/saloev/javascript-algorithms-and-data-structures)


### Degree

#### _01-09-2015 - 10-07-2019_ 

-----
Bachelor's Degree in Saint Petersburg Electrotechnical University "LETI"

Mathematics and Computer Science

-----

#### _01-09-2019 - 10-07-2022_ 

-----
Master's Degree in Saint Petersburg Electrotechnical University "LETI"

Software engineering

-----

----

## English
----
I started learning English at school, but not use it at all(unfortunately) Also at university for 2 year i learned english and my proninsiation gets better. 

_Now my English level is Pre Intermediate._

----


