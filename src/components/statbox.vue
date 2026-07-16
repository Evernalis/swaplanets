<script>
import planicon from './planicon.vue'
export default {
    components: {
        planicon
    },
    props: {
        planetdata: Object,
        highlighted: String,
        selectable: Boolean,
        locked: Boolean,
        mystery: Boolean,
        overridehighlight: String,
        rolling: Boolean
    },   
    data() {
        return {
            desctemplate: '!!PLANETNAME is a!!SIZE planet with a?? !!CLIMATE climate.',
            descript: undefined,
            validcats: ['diameter', 'orbital_period','rotation_period','surface_water','population', '   '],
            transkey: {
            'name':'This shouldn\'t be here.',
            'rotation_period': 'Day length',
            'orbital_period':'Orbit length',
            'diameter':'Diameter',
            'climate':'Climate',   
            'gravity':'Relative planet gravity',
            'terrain':'Terrain types',
            'surface_water':'Percentage of water cover',
            'population':'Population in millions',
            'residents': 'Residents',
            'films':'Featured in',
            'created':'This shouldn\'t be here.',
            'edited':'This shouldn\'t be here.',
            'url':'This shouldn\'t be here.'
        },
                choice: undefined,
                planetdat: {
                    'name': 'Hidden Planet',
                    'rotation_period': 'unknown',
                    'orbital_period': 'unknown',
                    'diameter': 'unknown',
                    'climate': 'NULL',
                    'terrain': 'undiscovered',
                    'surface_water': 'unknown',
                    'population': 'unknown'
                },
                pName: 'error',


        }
    },
    methods: {
        onNewPlanet(){
            console.log(this.planetdata);
            if (this.planetdata != undefined) {
            this.planetdat={
                'name':this.planetdata.name,
                'rotation_period':this.planetdata.rotation_period,
                'orbital_period':this.planetdata.orbital_period,
                'diameter':this.planetdata.diameter,
                'climate':this.planetdata.climate,
                'terrain':this.planetdata.terrain,
                'surface_water':this.planetdata.surface_water,
                'population':this.planetdata.population,
            }
            if (this.rolling) {
                this.planetdat.name='...';
                this.planetdat.terrain='undiscovered';
            }
        }

            if (this.planetdat==undefined || this.mystery) {
                this.planetdat = {
                    'name': 'Hidden Planet',
                    'rotation_period': 'unknown',   //duplicates data so values can be changed without altering source data
                    'orbital_period': 'unknown',
                    'diameter': 'unknown',
                    'climate': 'NULL',
                    'terrain': 'undiscovered',
                    'surface_water': 'Unknown',
                    'population': 'unknown'
                }
            }
            if  (this.planetdat.surface_water =='unknown') { // 'unknown' is too wide and causes card issues with surface water, so is swapped with 0
                this.planetdat.surface_water == 0 ;
                alert(this.planetdat.surface_water);
            }
            if(this.rolling) {
                this.planetdat.climate='NULL';
                this.planetdat.name='???';
                this.descript='Opponent is deciding...'
                return;
            }
            this.pName=this.planetdat['name'];
            let psize;
            if (this.planetdat.climate=='NULL') {
                this.descript='Make your selection to reveal this planet!'  
            } else {
            if (this.pName=='Bespin'){psize='n immense gas'}
            else if (this.planetdat.diameter <9000){psize=' small'}
            else if (this.planetdat.diameter < 12000){psize=' moderately sized'}
            else {psize=' large'}
            let grammarfix='';
            if (['a','e','i','o','u'].includes(this.planetdat.climate.split(', ')[0][0])){grammarfix='n'} //adds 'n' before a climate with a vowel start
            this.descript=this.desctemplate.replace('!!PLANETNAME', this.pName).replace('!!SIZE', psize).replace('??', grammarfix).replace('!!CLIMATE', this.planetdat.climate.split(', ')[0])
            }}
    },
    watch: {
        planetdata: {
        handler() {this.onNewPlanet()}, immediate: true}
    ,
        mystery:{
        handler() {this.onNewPlanet()}},
        locked: {handler(newv) {if(newv) {this.choice=undefined}}}, // unlock means new round, so reset choice
        rolling: {handler() {this.onNewPlanet()}}   
    }
} 
</script>

<template>
    <div class="statbox" >

                <planicon style="display: inline; margin-right: 155px;":terraintypes="planetdat.terrain"/>

                <h2 style="display:inline-block;">{{ pName }}</h2>
                <p style="display:inline-block; margin-left: 155px;">{{descript}}</p>
                <ul id="statbox" ">
                    <li class="listitem" v-for="(data,cat) in planetdat" v-show="validcats.includes(cat)" :class="{ highlight: choice==cat|| locked&&overridehighlight==cat, pullback: locked&& ![overridehighlight, choice].includes(cat) || mystery}">
                        {{ transkey[cat] }}: {{ data }}  <button :class="{highlight: choice==cat, pullback: cat != choice && locked, vanish:locked||(!locked&&choice==cat)}" v-if="selectable" @click="if(!locked){choice=cat;console.log(choice); $emit('upate', choice)}" :id="'select_'+cat" class="selector" > > </button>
                    </li>
                </ul>
            </div>
</template> 