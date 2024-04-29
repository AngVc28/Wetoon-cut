# Wetoon-cut
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Image Crop and Download</title>
</head>
<body>

<h2>Upload dan Crop Gambar</h2>

<!-- Form untuk upload gambar -->
<form id="uploadForm" enctype="multipart/form-data">
    <input type="file" id="imageInput" accept="image/*" required>
    <button type="submit">Upload</button>
</form>

<hr>

<!-- Area untuk menampilkan gambar yang diupload -->
<div id="imagePreview"></div>

<!-- Tombol untuk mendownload gambar yang sudah di-crop -->
<button id="downloadButton" style="display:none;">Download Gambar</button>

<!-- Area untuk menampilkan hasil gambar yang sudah di-crop -->
<div id="croppedImages"></div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/jszip/3.5.0/jszip.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/FileSaver.js/2.0.5/FileSaver.min.js"></script>
<script>
document.getElementById('uploadForm').addEventListener('submit', function(event) {
    event.preventDefault(); // Mencegah submit form

    var fileInput = document.getElementById('imageInput');
    var imagePreview = document.getElementById('imagePreview');
    var downloadButton = document.getElementById('downloadButton');

    // Mengecek apakah file yang diupload adalah gambar
    if (fileInput.files && fileInput.files[0]) {
        var reader = new FileReader();
        reader.onload = function(e) {
            var img = new Image();
            img.src = e.target.result;

            // Menampilkan gambar yang diupload
            img.onload = function() {
                imagePreview.innerHTML = '<img src="' + img.src + '" id="uploadedImage">';
                downloadButton.style.display = 'block';
            };
        };
        reader.readAsDataURL(fileInput.files[0]);
    }
});

document.getElementById('downloadButton').addEventListener('click', function() {
    var uploadedImage = document.getElementById('uploadedImage');
    var canvas = document.createElement('canvas');
    var ctx = canvas.getContext('2d');

    // Mengatur ukuran canvas sesuai dengan yang diinginkan (800x1280px)
    canvas.width = 800;
    canvas.height = 1280;

    // Mendapatkan rasio gambar yang diunggah
    var aspectRatio = uploadedImage.width / uploadedImage.height;

    // Menghitung jumlah gambar yang akan dihasilkan
    var numImages = Math.ceil(uploadedImage.height / canvas.height);

    // Mengatur array untuk menampung potongan gambar
    var croppedImages = [];

    // Area untuk menampilkan hasil gambar yang sudah di-crop
    var croppedImagesDiv = document.getElementById('croppedImages');
    croppedImagesDiv.innerHTML = ''; // Membersihkan hasil sebelumnya (jika ada)

    // Memotong dan menyimpan setiap gambar ke dalam array
    for (var i = 0; i < numImages; i++) {
        var startY = i * canvas.height;
        var cropHeight = canvas.height;

        // Menyimpan sisa crop jika ini adalah potongan terakhir
        if (i === numImages - 1 && uploadedImage.height % canvas.height !== 0) {
            cropHeight = uploadedImage.height % canvas.height;
        }

        // Memeriksa apakah perlu membuat gambar baru jika lebarnya lebih besar dari 800px
        if (aspectRatio > 1) {
            var cropWidth = canvas.width;
            var newHeight = cropWidth / aspectRatio;

            // Memotong gambar
            canvas.height = newHeight;
            cropHeight = Math.min(canvas.height, cropHeight); // Menyamakan cropHeight dengan tinggi gambar baru
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = 'white'; // Mengisi area yang tidak terpakai dengan warna putih
            ctx.fillRect(0, 0, canvas.width, canvas.height);
            ctx.drawImage(uploadedImage, 0, startY, canvas.width, cropHeight, 0, 0, canvas.width, cropHeight);
        } else {
            var cropWidth = uploadedImage.width;
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = 'white'; // Mengisi area yang tidak terpakai dengan warna putih
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Menambahkan kode untuk menghilangkan bagian sisa
            var startX = (canvas.width - cropWidth) / 2;
            ctx.drawImage(uploadedImage, 0, startY, cropWidth, cropHeight, startX, 0, cropWidth, cropHeight);
        }

        // Menampilkan hasil gambar yang sudah di-crop
        var img = document.createElement('img');
        img.src = canvas.toDataURL('image/jpeg');
        img.classList.add('cropped-image');
        croppedImagesDiv.appendChild(img);

        // Menambahkan gambar ke dalam array
        croppedImages.push(canvas.toDataURL('image/jpeg').replace(/^data:image\/(png|jpeg);base64,/, ''));
    }

    // Membuat file zip
    var zip = new JSZip();
    croppedImages.forEach(function(imgData, index) {
        zip.file('gambar_' + (index + 1) + '.jpg', imgData, { base64: true });
    });

    // Mendownload file zip
    zip.generateAsync({ type: 'blob' }).then(function(content) {
        saveAs(content, 'gambar_crop.zip');
    });
});
</script>

</body>
</html>
