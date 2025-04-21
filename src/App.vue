<script setup>
import { ref } from 'vue'
import pako from 'pako';

const mapcode= ref("AA")
const decompressedData =ref(Uint8Array.from(0))

const decodemap= ref("decodemap here")

const version= ref(0)
const title= ref("Title")
const description= ref("Description")
const author= ref("Author")

const err= ref(false)

function readint(decompressedData, length,index ) {
  const buffer = decompressedData.buffer;
  const dataView = new DataView(buffer);

  let result = 0n; // 使用 BigInt 来存储可能的大数
  for (let i = 0; i < length; i++) {
    // 读取每个字节并左移相应的位数，然后累加到结果中
    result |= (BigInt(dataView.getUint8(index + i)) << (BigInt(i) * 8n));
  }

  return {
    combinedInt: Number(result),
    lastBitOffset: index+length
  };
}

function readstr(decompressedData,index ) {
  let idx=index;
  let length  = decompressedData[idx];
  if (length >= 128){
    idx += 1;
    if (decompressedData[idx] == 2){
      length = 256;
    }
  }       

  // 选取部分数据
  const subArray = decompressedData.slice(idx+1, idx+1 + length);

  // 使用TextDecoder来解码Uint8Array到字符串
  const textDecoder = new TextDecoder('utf-8');
  const decodedString = textDecoder.decode(subArray);

  console.log("readstr")
  console.log(idx)
  console.log(subArray)
  console.log(decodedString)

  return {
    combinedStr: decodedString,
    lastBitOffset: idx+1 + length
  };
}

function decode(){

  try{
    decodemap.value=atob(mapcode.value);

    // 假设compressedString是Base64编码的，首先需要将其转换为Uint8Array
    const compressedData = Uint8Array.from(decodemap.value, c => c.charCodeAt(0));

    // 使用pako解压
    decompressedData.value = pako.inflate(compressedData);

    let index = 0;
    let numContent= {
    combinedInt: 0,
    lastBitOffset: 0
  };
    let strContent={
    combinedStr: "",
    lastBitOffset: 0
  };

    numContent=readint(decompressedData.value,4,index);
    version.value= numContent.combinedInt;
    index = numContent.lastBitOffset;
    strContent=readstr(decompressedData.value,index);
    title.value= strContent.combinedStr;
    index = strContent.lastBitOffset;

    console.log("index")
    console.log(index);
    console.log("numContent")
    console.log(numContent);
    console.log("strContent")
    console.log(strContent);

  }
  catch(e){
    decodemap.value=mapcode.value;
    console.error('Failed to decode base64 string', e);
    err.value= true;
  }
}


</script>

<template>
  <button @click="decode">Decode map</button>
  <input v-model="mapcode">
  <!-- <p v-if="!err">{{ decodemap }}</p> -->
  <!-- <p v-if="!err">{{ decompressedData }}</p> -->
  <p v-if="!err">版本：{{ version}}</p>
  <p v-if="!err">地图名：{{ title}}</p>
  <p v-if="err">Failed to decode base64 string </p>
</template>

<style scoped>
button {
  font-weight: bold;
}
</style>