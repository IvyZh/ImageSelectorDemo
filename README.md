# ImageSelectorDemo

仿微信的图片加载案例

http://blog.csdn.net/lmj623565791/article/details/39943731/



* 尽可能的避免OOM
	* LruCache
	* 根据图片显示大小来压缩图片
* 用户操作UI控件必须充分流程
	* getView里面尽可能bu做耗时操作
* 用户预期显示的图片尽可能的快
	* LIFO



Task-->run()

1. 获得图片显示的大小
2. 使用options
3. 加载图片并防暑Lrucache



后台轮询线程-> Task->线程池去执行



异步消息处理框架：Handler Lopper Message

步骤：


* 新建类ImageLoader
	* 单一实例
	* 成员变量 
		* LruCache<String,Bitmap> mLruCache;
		* ExecutorService mThreadPool;
		* int default_thread_count = 1;
		* enum Type{ FIFI,LIFO}
		* LinkedList<Runnable> mTaskQueue;//人去队列
		* Type mType = Type.LIFO;
		* Handler mUIHandler;


* 变量初始化

	
	private ImageLoad(int threadCount,Type type){
	
		mPoolThread = new Thread(){
		
			@Ovidede
			run(){
				Looper.prapare();
				mPoolThreadHandler = new Handler(){
					handleMessage(Message msg){
						// 线程池取出一个任务进行执行
						mThreadPool.excute(getTask());

						mSemphoreThreadPool.acquire();
					}
				}

			mSemphorePoolThreadHandler.release();
			
			Looper.loop();
			}
		}	
		mPoolThread.start();

		// 获取应用最大可用内存
		int maxMemery = (int) Runtime.getRuntime().maxMemory();
		int cacheMemory = maxMemory / 8;
		
		mLruCache = new LruCache<String , BItmap>(cacheMemory){
		
			int sizeOf(String key,Bitmap value){
				return value.getRowBytes() * value.getHeigth();
			}
		}

		mThreadPool = Exectutors.newFixedThreadPool(threadCount);
		
		mTaskQueue = new LinkedList<Runnable>();
	
		mType = type;


		// 初始化mSemphoreThreadPool
		mSemphoreThreadPool = new mSemphoreThreadPool(threadCount);

	}


* 初始化 mUIHandler

public void loadImage(String path,ImageView imageView){
	
	imageView.setTag(path);
	if(mUIHandler==null){
		mUIHandler = new Handler(){
			handle(Message msg){
			ImageBeanHolder holder = (ImageBeanHolder)msg.obj;
			Bitmap bm = holder.bm;
			String path = holder.path;
			ImageView imangeView = holder.imageView;
			if(imageView.getTag().equeal(path)){
				imageView.setImageBitmap(bm);
			}
		
		}
		};	
	}

	Bitmap bm = getBitmapFromLruCache(path)();
	if(bm!=null){
		Message msg = Message.obtain();
		ImageBean holder  = new ImageBeanHolder();
		holder.imageView = imageView;
		ho.path = path;
		ho.bitmap = bm;
		msg.obj = holder;
		mUIHandler.send(msg);	
	}
}

getBitmapFromLruCache(String path){
	return mLruCache.get(path)
}


ImageBeanHolder{
	String path;
	Bitmap bm;
	ImageView imageView;
}


* 获取图片需要显示的大小


	if(bm!=null){
		Message msg = Message.obtain();
		ImageBean holder  = new ImageBeanHolder();
		holder.imageView = imageView;
		ho.path = path;
		ho.bitmap = bm;
		msg.obj = holder;
		mUIHandler.send(msg);	
	}else{
		addTasks(new Runnalble(){
		// 加载图片
		// 1. 获得图片显示的大小
		ImageViewSize size = getImageViewSize(imageView);
		});	
		// 2. 压缩图片
		Bitmap bm = decodeSampledBitmapFromPath(path,size.w,size.h);

		// 3. 把图片加入到缓存
		addBitmapToLruCache(path,bm);


		refreshBitmap(path,imageview,bm);

		mSemphoreThreadPool.release();

	}

 synchronize addTasks(Runnalbe r){
	mTaskQueue.add(r);

	//if(mPoolThreadHandler ==null) wait();

	if(mPoolThreadHandler ==null)
		mSemphorePoolThreadHandler.acquire();

	mPoolThreadHandler.sendEmptyMessage(0x110);
}




Runnable getTask(){
	if(mType == Type.FIFO){
		return mTask.Queus.removeFirst();
	}else{
	 return mTask.Queus.removeLast();
	}
}

// 根据imageview获取适当的压缩宽和高

	ImageViewSize getImageViewSize(ImageView imageView){
		ImageViewSize size = new ImageViewSize();
	
		LayoutParams lp = imageView.getLayParams();
		
		int w = imageView.getWidth();
		
		//int w = (lp.width == LayoutParams.Wrap_conten?0:imageView.getWidth());//ViewGroup的
	
		if(w<=0){
			w = lp.width;
		}
		
		if(w<0){
			w = imageView.getMaxWidth();//TODO api16	
		}
	
		if(w<0){
			w = display.widthPixels();
		}

		// heigth 略。

		size.w = w;
		size.h = h;
		
		return size;
	
	}

ImageViewSize {
	int h;
	int w;
}


* 压缩图片

Bitmap decodeSampldecodeSampledBitmapFromPathedBitmapFromPath(String path,int w,int h){
	Bitmap.options op = new BitmapFactory.Options();
	op.inJust = true;
	Bitmap.decodeFile(path,op);
	
	op.inSimpleSize = caculateInSampleSize(op,w,h);

	// 使用inSimleSize 解析图片
	op.inJsutDecodeBound = false;
	
	return Bitmap.decodeFile(path,op);

}

int caculateInSampleSize(Options op,int w,int h){
		int width = op.outWidth；
		int heigth = op.outHeight();

		int inSampleSize = 1;
	
		if(width >q || heigth >h){
			int wr = Math.round(width * 1.0f /w);
			int hr = Math.round(heigth * 1.0f /h);

			inSampleSize = Math.max(wr,hr);
		}
		return inSimpleSize;
}


* 后台轮询线程的并行

addBitmapToLruCache(String path,Bitmap bm){
	if(getBitmaoFromLruc(path)!=null){
		if(bm!=null){
			mLruCache.put(path,bm);
		}
	}
}


refreshBitmap(){

	Message msg = Message.obtain();
	ImageBean holder  = new ImageBeanHolder();
	holder.imageView = imageView;
	ho.path = path;
	ho.bitmap = bm;
	msg.obj = holder;
	mUIHandler.send(msg);	
}


* 并行。

private Semaphore mSemphorePoolThreadHandler = new Semaphore(0);

* 利用信号量控制任务队列


private Semaphore mSemphoreThreadPool ;


* 反射获取宽高最大值

getMaxHeigth();--》


int getImageViewFieldValue(Object obj,String filedName)
{
	int v = 0;
	Field filed = ImageView.class.getDecalse(filedName);
	int fieldValue = field.getInt(obj);
	if(filedValie>0 && fieldValue<Max.INterge){
		v = filedValue;
	}
	return v;
}

> mMaxHeigth  mMaxWidth


* 主布局

GridView
	
listSeletor
spacingstretchMode clumnWidth


* 扫描图片ContentProvider
* 使用Hanlder晚上扫描图片
* 适配器


* setColorFilter