```
npm i multer
npm i -D @types/multer
```

```ts
import multer from 'multer'
import path from 'path'

const storage = multer.diskStorage({
  destination(req, file, cb) {
    cb(null, 'uploads/')
  },
  filename(req, file, cb) {
    cb(null, `${file.fieldname}-${Date.now()}${path.extname(file.originalname)}`)
  },
})

function checkFileType(file: fileInterface, cb: Function) {
  const filetypes = /jpg|jpeg|png/
  const extname = filetypes.test(path.extname(file.originalname).toLowerCase())
  const mimetype = filetypes.test(file.mimetype)

  if (extname && mimetype) {
    return cb(null, true)
  } else {
    cb('Images only!')
  }
}

const upload = multer({
  storage,
  limits: { fileSize: 1000000 },
  fileFilter: function (req, file, cb) {
    checkFileType(file, cb)
  },
})

interface fileInterface {
  fieldname: string
  originalname: string
  mimetype: string
}

export default upload
```

```ts
// create a folder named uploads in root as well
app.post('/upload', upload.single('image'), (req: Request, res: Response) => {
  res.status(404).json({ message: 'Done' })
})

app.post('/multiple', upload.array('image', 3), (req, res) => {
  res.status(404).json({ message: 'Done' })
})
```
