```ts
import { statSync, createReadStream } from 'fs'

app.get('/video', (req: Request, res: Response) => {
  const range = req.headers.range as string
  const videoPath = './uploads/1.mp4'
  const videoSize = statSync(videoPath).size

  const chunkSize = 500 * 1e5 // how many data we want to send per chunk
  const start = Number(range.replace(/\D/g, ''))
  const end = Math.min(start + chunkSize, videoSize - 1)

  const contentLength = end - start + 1

  const headers = {
    'Content-Range': `bytes ${start}-${end}/${videoSize}`,
    'Accept-Ranges': 'bytes',
    'Content-Length': contentLength,
    'Content-Type': 'video/mp4',
  }
  res.writeHead(206, headers)

  const stream = createReadStream(videoPath, { start, end })
  stream.pipe(res)
})
```

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>

  <body>
    <video src="http://localhost:5000/video" width="1080" controls></video>
  </body>
</html>
```
