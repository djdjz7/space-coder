<script setup lang="ts">
import { ref, watch } from "vue";

const mapping_from_cipher = new Map<number, number>([
  [32, 0],
  [8192, 1],
  [8193, 2],
  [8194, 3],
  [8195, 4],
  [8196, 5],
  [8198, 6],
  [8199, 7],
]);

const mapping_to_cipher = [32, 8192, 8193, 8194, 8195, 8196, 8198, 8199];

const cipher = ref("");
const plain = ref("");
const encoder = new TextEncoder();
const decoder = new TextDecoder();
const encode = (plain: string) => {
  const buffer = encoder.encode(plain);
  const encoded = [];
  for (const byte of buffer) {
    encoded.push(mapping_to_cipher[Math.floor(byte / 64)]);
    encoded.push(mapping_to_cipher[Math.floor((byte % 64) / 8)]);
    encoded.push(mapping_to_cipher[byte % 8]);
  }
  return "sp" + String.fromCharCode(...encoded) + "ce";
};

const decode = (cipher: string) => {
  const buffer = [];
  for (let i = 2; i < cipher.length - 2; i += 3) {
    const byte =
      mapping_from_cipher.get(cipher.charCodeAt(i))! * 64 +
      mapping_from_cipher.get(cipher.charCodeAt(i + 1))! * 8 +
      mapping_from_cipher.get(cipher.charCodeAt(i + 2))!;
    buffer.push(byte);
  }
  return decoder.decode(new Uint8Array(buffer));
};

watch(plain, (value) => {
  cipher.value = encode(value);
});

watch(cipher, (value) => {
  plain.value = decode(value);
});
</script>

<template>
  <h1>Space Coder</h1>
  <textarea placeholder="明文" v-model.lazy="plain"></textarea>
  <textarea placeholder="密文" v-model.lazy="cipher"></textarea>
</template>

<style scoped>
textarea {
  width: 100%;
  height: 100px;
}
</style>
