<template>
  <div>
  <div>{{this.stockname + this.code}}</div>
  <div><highcharts :constructor-type="'stockChart'" :options="stockOptions"></highcharts></div>
  </div>
</template>
<script>
import Highcharts from 'highcharts'
import { data } from './stockData.js'
const stockDataddd = data.map(d => d.slice(0.2))
export default {
  data() {
  return {
      stockOptions: {
        rangeSelector: {
          selected: 1,
          inputDateFormat: '%Y-%m-%d'
        },
        title: {
          text: ""
        },
        plotOptions: {
            series: {
                states: {
                    hover: {
                        enabled: false
                    }
                }
            }
        },
        xAxis: {
          dateTimeLabelFormats: {
            millisecond: "%H:%M:%S.%L",
            second: "%H:%M:%S",
            minute: "%H:%M",
            hour: "%H:%M",
            day: "%m-%d",
            week: "%m-%d",
            month: "%y-%m",
            year: "%Y"
          },
          minRange: 3600000*24 *30,
          min: null,
          max: null,
        },
        tooltip: {
          split: false,
          shared: true
        },
        yAxis: [
          {
            labels: {
              align: "left",
              x: -3
            },
            title: {
              text: "股价(元)"
            },
            height: "65%",
            resize: {
              enabled: true
            },
            lineWidth: 0,
            showLastLabel: true,
            showFirstLabel: true,
          },
          {
            labels: {
              align: "left",
              x: -3
            },
            title: {
              text: "成交量"
            },
            top: "65%",
            height: "35%",
            offset: 0,
            lineWidth: 2
          }
        ],
        series: [
          {
            type: "candlestick",
            name: "价格",
            color: "green",
            lineColor: "green",
            upColor: "red",
            upLineColor: "red",
            tooltip: {},
            navigatorOptions: {
              color: Highcharts.getOptions().colors[0]
            },
            data: [],
            dataGrouping:[],
            yAxis: 0,
            maxPointWidth: 5,
            id: "maindayk"
          },
          {
            type: "column",
            name: "成交量(亿)",
            data: [],
            yAxis: 1,
            dataGrouping: [],
            maxPointWidth: 7,
          },
          {
            type:'line',
            name:'ma5' ,
            data: [],
            color: "#fcfc54",
            yAxis: 0,
            dataGrouping:[],
            lineWidth: 1,
          },{
            type:'line',
            name:'ma10' ,
            data: [],
            color: "#dd54fc",
            yAxis: 0,
            dataGrouping:[],
            lineWidth: 1,
          },{
            type:'line',
            name:'ma20' ,
            data: [],
            color: "#54fc54",
            yAxis: 0,
            dataGrouping:[],
            lineWidth: 1,
          },{
            type:'line',
            name:'ma30' ,
            data: [],
            color: "#3f54fc",
            yAxis: 0,
            dataGrouping:[],
            lineWidth: 1,
          },
        ],
      }, 
      code: "",
      stockname: "",
      dayk:[],
      reladayk:[],
    };
  },

  methods: {
    createData() {
      let stockData = this.dayk
      let dataLength = stockData.length;
      let groupingUnits = [
        [
          "week", // unit name
          [1] // allowed multiples
        ],
        ["month", [1, 2, 3, 4, 6]]
      ];
      let i = 0;
      let timeStamp = 0;
      let amountv = 0;
      let ohlc = [];
      let dayday = [];
      let reladayday = [];
      let volume = [];
      let maset = [5,10,20,30];
			let ma = [];
      for (i = 0; i < dataLength; i += 1) {
        timeStamp = stockData[i][0];
        ohlc.push([
          timeStamp, // the date
          stockData[i][1], // open
          stockData[i][2], // high
          stockData[i][3], // low
          stockData[i][4] // close
        ]);
        volume.push({
          x: timeStamp,
          y: stockData[i][5], // the date
          color: stockData[i][1] > stockData[i][4] ? 'green' : 'red'
        });
        for (let j = 0; j < maset.length; j++) {
          let value = maset[j];
          if(typeof ma['ma'+value] == "undefined"){
            ma['ma'+value]=[];
          }
          if(typeof ma[value+'total'] == "undefined"){
            ma[value+'total']=0;
          }
          if(i < value)
          {
            ma[value+'total'] += stockData[i][4];
            ma['ma'+value].push([timeStamp,null]);
          } else {
            ma[value+'total'] += (stockData[i][4] - stockData[i - value][4]);
            let kk = Number((ma[value+'total']/value).toFixed(2))
            ma['ma'+value].push([timeStamp, kk]);
          }   
        }
      }

      if(dataLength < 20) {
        this.stockOptions.xAxis.min = ohlc[0][0]
        this.stockOptions.xAxis.max = ohlc[0][0] + 20 * 24 * 3600000

        let curtime = ohlc[dataLength - 1][0]
        for (i = dataLength; i < 20; i+=1) {
          curtime += 24 * 3600000
          console.log(curtime)
          volume.push({
          x: curtime,
          y: null, // the date
          color: 'white'
        });
        }
      }

      this.stockOptions.rangeSelector.selected = 1
      this.stockOptions.series[0].data = ohlc;
      this.stockOptions.series[1].data = volume;
      this.stockOptions.series[2].data = ma['ma5'];
      this.stockOptions.series[3].data = ma['ma10'];
      this.stockOptions.series[4].data = ma['ma20'];
      this.stockOptions.series[5].data = ma['ma30'];

      
    }
  },

  created() {
    this.code = 'sh600001';
    this.dayk = stockDataddd;
    this.stockname = '平安银行';
    this.createData();
  }
};
</script>
