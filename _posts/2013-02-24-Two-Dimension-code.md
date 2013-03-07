---
layout: post
title: 二维码的生成与解析
category : Android
tags : [QRcode]
---
{% include JB/setup %}


`QRcode encode and decode`  

### 1. QRcode encoder
使用[qrcode_swetake.jar](http://www.venus.dti.ne.jp/~swe/program/qrcode_java0.50beta10.tar.gz)生成二维码，
具体原理参见[How to create QRcode](http://swetake.com/qr/qr1_en.html)。  
具体示例如下：

	private static final Charset UTF_8 = Charset.forName("utf-8");
	
	public BufferedImage encode(String content, int size) throws Exception {
		if (content == null || content.isEmpty() || content.length() > 800) {
			throw new IllegalAccessError("content is illegal");
		}
		// 获得内容的字节数组，设置编码格式
		byte[] data = content.getBytes(UTF_8);
		Qrcode qrcode = new Qrcode();
		// 设置二维码排错率，可选L(7%)、M(15%)、Q(25%)、H(30%)，排错率越高可存储的信息越少，但对二维码清晰度的要求越小
		qrcode.setQrcodeErrorCorrect('M');
		qrcode.setQrcodeEncodeMode('B');
		// 设置设置二维码尺寸，取值范围1-40，值越大尺寸越大，可存储的信息越大
		qrcode.setQrcodeVersion(size);
		// 图片尺寸
		int imgSize = 67 + 12 * (size - 1);
		BufferedImage img = new BufferedImage(imgSize, imgSize, BufferedImage.TYPE_INT_RGB);
		Graphics2D graph = img.createGraphics();
		// 设置背景颜色
		graph.setBackground(Color.WHITE);
		// 设定图像颜色> BLACK
		graph.setColor(Color.BLACK);
		graph.clearRect(0, 0, imgSize, imgSize);
		// 设置偏移量，不设置可能导致解析出错
		int pixoff = 2;
		// 输出内容> 二维码
		boolean[][] codeOut = qrcode.calQrcode(data);
		for (int i = 0; i < codeOut.length; i++) {
			for (int j = 0; j < codeOut.length; j++) {
				if (codeOut[j][i]) {
					graph.fillRect(j * 3 + pixoff, i * 3 + pixoff, 3, 3);
				}
			}
		}
		graph.dispose();
		img.flush();
		return img;
	}

转自[java二维码生成与解析代码实现](http://blog.csdn.net/about58238/article/details/7494704)，稍有修改。  

### 2. QRcode decoder
使用[qrcode.jar](http://sourceforge.jp/projects/qrcode/)解析二维码。  
示例代码如下：

	public String decode(InputStream input) throws Exception {
		if (input == null) {
			throw new IllegalArgumentException("input is required");
		}
		BufferedImage img = ImageIO.read(input);
		QRCodeDecoder decoder = new QRCodeDecoder();
		byte[] data = decoder.decode(new QRcodeImage(img));
		return new String(data, UTF_8);
	}
	
	class QRcodeImage implements QRCodeImage {

		BufferedImage image;

		public QRcodeImage(BufferedImage source) {
			this.image = source;
		}

		public int getWidth() {
			return image.getWidth();
		}

		public int getHeight() {
			return image.getHeight();
		}

		public int getPixel(int x, int y) {
			return image.getRGB(x, y);

		}
	}


### references
+ [How to create QRcode](http://swetake.com/qr/index-e.html)
+ [Sourceforge QRcode](http://sourceforge.jp/projects/qrcode/)
+ [java二维码生成与解析代码实现](http://blog.csdn.net/about58238/article/details/7494704)
+ [Java实现二维码QRCode的编码和解码](http://blog.csdn.net/chenhuade85/article/details/7492985)
+ [二维码的生成](http://shenjun134.iteye.com/blog/1662044)

