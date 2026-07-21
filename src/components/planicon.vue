<script>
export default {
    inject: ['biolist'],   //list of all unique biome types
    data() {
        return{
        biometype: [],              
        biomekey: [                                 // strings at index 1 are actual file name components
            [[0,19,20,22,42,43,46],'Desert'],//desert
            [[1,13,32,34,51],'Grass'], //grass
            [[2,7],'Mountain'], //mountain
            [[3,4,9,11],'Forest'], //forest
            [[5,6,29,30],'Ice'], //ice
            [[8,14,38],'Swamp'], //swamp
            //[12, 27],], //lakesandrivers    UNUSED
            [[15,35],'Urban'], //urban
            [[16,37,41],'Ocean'], //ocean
            [[24,25,48],'Lava'], //lava
            [[40, 45],'Islands'], //islands
            [[10],'Bespin'], //gasgiant
            [[28],'Asteroid'], //asteroid
            [[31],'Fungus'] //fungus,
            
        ]
    }},
    props: {
        terraintypes: String   ///// split into array later 
    },
    watch: {
        terraintypes: {
        handler() {this.generatePlanetIcon()}, immediate: true}
    },
     methods: {
             generatePlanetIcon() {
                console.log('!')
                    let terrtypes=this.terraintypes;
                    let biodat=[]; //list of biome codes in planet
                    this.biometype=[];
                    let rawdat=terrtypes.split(', ');
                    for(let terrain of rawdat){
                        biodat.push(this.biolist.indexOf(terrain))
                    }
                    console.log('bio' +biodat);
                    for (let terrcode of biodat) {
                        for (let mateType of this.biomekey) {
                            if (mateType[0].includes(terrcode)){
                                this.biometype.push(new URL('.assets/planetimage/material'+mateType[1]+'.png',import.meta.url).href) 
                            } //material URLs follow a common format, this fits the material into the template and is used directly by segment
                        }
                    }
                    if(this.biometype.length==0) {this.biometype[0]=new URL(`.assets/planetimage/blankplanet.png`, import.meta.url).href   } // 0 biomes means planet is not shown yet or an error has occured
                    if(this.biometype.length==1) {this.biometype[1]=this.biometype[0]};         // fills biome data for <4 biomes
                    if(this.biometype.length==2) {this.biometype[2]=this.biometype[1]};
                    if(this.biometype.length==3) {this.biometype[3]=this.biometype[0]};
                },
        }
}
</script>

<template>
    <div id="graphicbox" @click="generatePlanetIcon()">
                    <img class="planico" src="./planetimage/masktemplate.png">
                    <img class="planico sec0" :src="biometype[0]">
                    <img class="planico sec1" :src="biometype[1]">
                    <img class="planico sec2" :src="biometype[2]">
                    <img class="planico sec3" :src="biometype[3]">
                </div>
</template>

<style scoped>
    .sec0 {
        mask-image: url(./src/components/planetimage/section0.png);
    }
    .sec1 {
        mask-image: url(./src/components/planetimage/section1.png);
    }
    .sec2 {
        mask-image: url(./src/components/planetimage/section2.png);
    }
    .sec3 {
        mask-image: url(./src/components/planetimage/section3.png);
    }
    .planico {
        width: 128px;
        height: 128px;
        position:fixed;
        mask-repeat: no-repeat;
        margin-left: 15px;
    }
</style>