import express from "express";
import multer from "multer";
import fs from "fs";
import path from "path";
import cors from "cors";

const app = express();
const PORT = process.env.PORT || 3000;
const PASSWORD = process.env.UPLOAD_PASSWORD || "elora";
const IMAGE_DIR = "./uploads";

// Ensure the uploads directory exists
if (!fs.existsSync(IMAGE_DIR)) {
  fs.mkdirSync(IMAGE_DIR);
}

// Multer config
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, IMAGE_DIR);
  },
  filename: (req, file, cb) => {
    cb(null, `${Date.now()}-${file.originalname}`);
  },
});

const upload = multer({ storage });

app.use(cors());
app.use(express.json());
app.use(express.static("public"));
app.use("/uploads", express.static(IMAGE_DIR));

// Password check middleware
function checkPassword(req, res, next) {
  const password = req.headers["x-password"];
  if (password !== PASSWORD) {
    return res.status(401).json({ error: "Unauthorized" });
  }
  next();
}

app.post("/upload", checkPassword, upload.single("image"), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: "No file uploaded" });
  }
  res.json({ filename: req.file.filename, url: `/uploads/${req.file.filename}` });
});

app.get("/images", checkPassword, (req, res) => {
  fs.readdir(IMAGE_DIR, (err, files) => {
    if (err) return res.status(500).json({ error: "Failed to list images" });
    const urls = files.map((file) => ({ filename: file, url: `/uploads/${file}` }));
    res.json(urls);
  });
});

app.delete("/images/:filename", checkPassword, (req, res) => {
  const filePath = path.join(IMAGE_DIR, req.params.filename);
  fs.unlink(filePath, (err) => {
    if (err) return res.status(404).json({ error: "File not found" });
    res.json({ success: true });
  });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
