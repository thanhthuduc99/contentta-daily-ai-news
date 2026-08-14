# Nhac nen cho Shorts

Repo khong kem san file nhac vi ly do ban quyen. Thu muc nay de trong, ban tu bo file MP3 cua minh vao.

## Yeu cau dinh dang

- **Dinh dang**: MP3 hoac WAV (khong dung OGG, FLAC)
- **Thoi luong**: 30-90 giay, loop duoc (diem cuoi noi mach voi diem dau)
- **Loai**: Chi nhac instrumental — khong co loi hat
- **Volume khuyen nghi**: 0.12 - 0.18 (trong data-volume cua audio element). Nhac nen chi de ho tro giong noi, khong duoc at giong

## Cach su dung

Dat file nhac vao thu muc `music/` nay. Khi build composition, tham chieu bang duong dan tuong doi:

```html
<audio src="../../music/ten-file.mp3"
       data-start="0"
       data-duration="30"
       data-volume="0.15"
       data-track-index="10"
       loop>
</audio>
```

## Luu y

- Nhac co ban quyen Creative Commons hoac royalty-free
- Nen chuan bi 2-3 file nhac voi energy khac nhau: calm, upbeat, dramatic
- Test volume bang `npx hyperframes preview` truoc khi render — dam bao giong noi luon ro rang hon nhac nen
- Khong dung nhac co beat qua manh o giay dau tien — de hook bang giong noi truoc
