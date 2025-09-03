# Introduction
Welcome to the snippet world .
this is the snippet code

- [memManager](https://github.com/cheldon-cn/): a memory manager 

- [x86_64-w64-mingw32 ](https://github.com/cheldon-cn/): configure items for different host
  
# 1. memManager

```
#include <vector>

//memory array
class CMemArray
{
public:
	CMemArray(){}
	~CMemArray(){}

public:
	int Add(char * pItem){
		m_list.push_back(pItem);
	}
	long Size(){
		return m_list.size();
	}
	int Set(long index, char* pItem){
		m_list.insert(index,pItem);
	}
	char* operator[](int index){
		return m_list[index];
	}

private:
	std::vector<char *> m_list;
};

//memory manager
class CMemManager
{
public:
	CMemManager();
	~CMemManager();

public:
	void *Alloc(int size);      //allocate buffer
	long Free();				//re goback to the start pos of the buffer 

private:
	CMemArray<char *>   m_list;
	char			   *m_buf;
	long				m_idx;
	int				    m_pos;
};


#define MEM_BLOCK_SIZE	(1024*1024*10)	//10M space
CMemManager::CMemManager(void)
{
	m_buf = NULL;
	m_pos = 0;        //record the pos of each memory block
	m_idx = 0;        //record the index of each memory block
}

CMemManager::~CMemManager(void)
{
	for (long i = 0; i < m_list.Size(); i++)
	{
		delete[]m_list[i];
	}
}

long CMemManager::Free()
{
	if (m_buf == NULL)return(1);

	m_buf = m_list[0];
	m_pos = 0;
	m_idx = 0;
	return(1);
}

void *CMemManager::Alloc(int size)
{
	if (m_buf == NULL)
	{
		m_buf = new char[MEM_BLOCK_SIZE];
		m_list.Add(m_buf);
	}

	// for linux or android,the memory buffer start position must be aligned, the default aligned byte is 4;
#if defined(BUTTER_LINUX) || defined(LINUX_ANDROID) || defined(BUTTER_IOS)
	if ((size % 4) == 0)
	{
		m_pos = (m_pos + 3) / 4 * 4;
	}
	else if ((size % 2) == 0)
	{
		m_pos = (m_pos + 1) / 2 * 2;
	}
#endif

	//m_buf
	if (MEM_BLOCK_SIZE < size + m_pos)
	{
		m_idx++;
		if (m_idx >= m_list.Size())
		{
			m_buf = new char[((MEM_BLOCK_SIZE > size) ? MEM_BLOCK_SIZE : size)];
			m_list.Add(m_buf);
		}
		else if (MEM_BLOCK_SIZE >= size)
			m_buf = m_list[m_idx];
		else
		{
			delete[]m_list[m_idx];
			m_buf = new char[size];
			m_list.Set(m_idx, m_buf);
		}
		m_pos = 0;
	}

	//allocate
	void *rtn = m_buf + m_pos;
	m_pos += size;
	return(rtn);
}

```

# 2. --host=x86_64-w64-mingw32 

```
--cygwin64
------lib
----------gcc
--------------i686-pc-cygwin
--------------i686-w64-mingw32
--------------x86_64-pc-cygwin
--------------x86_64-w64-mingw32
```

```
--msys64
------mingw64\lib\gcc
----------x86_64-w64-mingw32
------mingw32\lib\gcc
----------i686-w64-mingw32
```

```
--Android
------ndk-r20b\toolchains\llvm\prebuilt\windows-x86_64\lib\gcc
--------------aarch64-linux-android
--------------arm-linux-androideabi
--------------x86_64-linux-android
--------------i686-linux-android
```

```
./configure --host=i686-w64-mingw32     --prefix=/f/Codes/openSource/githubmaster/libiconv1.16/libiconv             CC=i686-w64-mingw32-gcc             CPPFLAGS="-I/usr/local/mingw32/include -Wall"             LDFLAGS="-L/usr/local/mingw32/lib"
```

```
PATH=/usr/local/mingw64/bin:$PATH
export PATH

./configure --host=x86_64-w64-mingw32 --prefix=/cygdrive/f/Codes/openSource/githubmaster/libiconv1.16/libiconv \
	CC=x86_64-w64-mingw32-gcc \
	CPPFLAGS="-I/usr/local/mingw64/include -Wall" \
	LDFLAGS="-L/usr/local/mingw64/lib"
	
```

# 3. arcgisonline url

https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/1/1/1

from  

https://github.com/giswqs/qgis-earthengine-examples/blob/master/Folium/ee-api-folium-setup.ipynb

```
from 

https://github.com/giswqs/qgis-earthengine-examples/blob/master/Basemaps/qgis_basemaps.py

sources = []
sources.append(["connections-xyz","Google Maps","","","","https://mt1.google.com/vt/lyrs=m&x=%7Bx%7D&y=%7By%7D&z=%7Bz%7D","","19","0"])
sources.append(["connections-xyz","Google Satellite", "", "", "", "https://mt1.google.com/vt/lyrs=s&x=%7Bx%7D&y=%7By%7D&z=%7Bz%7D", "", "19", "0"])
sources.append(["connections-xyz","Google Terrain", "", "", "", "https://mt1.google.com/vt/lyrs=t&x=%7Bx%7D&y=%7By%7D&z=%7Bz%7D", "", "19", "0"])
sources.append(["connections-xyz","Google Terrain Hybrid", "", "", "", "https://mt1.google.com/vt/lyrs=p&x=%7Bx%7D&y=%7By%7D&z=%7Bz%7D", "", "19", "0"])
sources.append(["connections-xyz","Google Satellite Hybrid", "", "", "", "https://mt1.google.com/vt/lyrs=y&x=%7Bx%7D&y=%7By%7D&z=%7Bz%7D", "", "19", "0"])
sources.append(["connections-xyz","Stamen Terrain", "", "", "Map tiles by Stamen Design, under CC BY 3.0. Data by OpenStreetMap, under ODbL", "http://tile.stamen.com/terrain/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "20", "0"])
sources.append(["connections-xyz","Stamen Toner", "", "", "Map tiles by Stamen Design, under CC BY 3.0. Data by OpenStreetMap, under ODbL", "http://tile.stamen.com/toner/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "20", "0"])
sources.append(["connections-xyz","Stamen Toner Light", "", "", "Map tiles by Stamen Design, under CC BY 3.0. Data by OpenStreetMap, under ODbL", "http://tile.stamen.com/toner-lite/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "20", "0"])
sources.append(["connections-xyz","Stamen Watercolor", "", "", "Map tiles by Stamen Design, under CC BY 3.0. Data by OpenStreetMap, under ODbL", "http://tile.stamen.com/watercolor/%7Bz%7D/%7Bx%7D/%7By%7D.jpg", "", "18", "0"])
sources.append(["connections-xyz","Wikimedia Map", "", "", "OpenStreetMap contributors, under ODbL", "https://maps.wikimedia.org/osm-intl/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "20", "1"])
sources.append(["connections-xyz","Wikimedia Hike Bike Map", "", "", "OpenStreetMap contributors, under ODbL", "http://tiles.wmflabs.org/hikebike/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "17", "1"])
sources.append(["connections-xyz","Esri Boundaries Places", "", "", "", "https://server.arcgisonline.com/ArcGIS/rest/services/Reference/World_Boundaries_and_Places/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "20", "0"])
sources.append(["connections-xyz","Esri Gray (dark)", "", "", "", "http://services.arcgisonline.com/ArcGIS/rest/services/Canvas/World_Dark_Gray_Base/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "16", "0"])
sources.append(["connections-xyz","Esri Gray (light)", "", "", "", "http://services.arcgisonline.com/ArcGIS/rest/services/Canvas/World_Light_Gray_Base/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "16", "0"])
sources.append(["connections-xyz","Esri National Geographic", "", "", "", "http://services.arcgisonline.com/ArcGIS/rest/services/NatGeo_World_Map/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "12", "0"])
sources.append(["connections-xyz","Esri Ocean", "", "", "", "https://services.arcgisonline.com/ArcGIS/rest/services/Ocean/World_Ocean_Base/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "10", "0"])
sources.append(["connections-xyz","Esri Satellite", "", "", "", "https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "17", "0"])
sources.append(["connections-xyz","Esri Standard", "", "", "", "https://server.arcgisonline.com/ArcGIS/rest/services/World_Street_Map/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "17", "0"])
sources.append(["connections-xyz","Esri Terrain", "", "", "", "https://server.arcgisonline.com/ArcGIS/rest/services/World_Terrain_Base/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "13", "0"])
sources.append(["connections-xyz","Esri Transportation", "", "", "", "https://server.arcgisonline.com/ArcGIS/rest/services/Reference/World_Transportation/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "20", "0"])
sources.append(["connections-xyz","Esri Topo World", "", "", "", "http://services.arcgisonline.com/ArcGIS/rest/services/World_Topo_Map/MapServer/tile/%7Bz%7D/%7By%7D/%7Bx%7D", "", "20", "0"])
sources.append(["connections-xyz","OpenStreetMap Standard", "", "", "OpenStreetMap contributors, CC-BY-SA", "http://tile.openstreetmap.org/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "19", "0"])
sources.append(["connections-xyz","OpenStreetMap H.O.T.", "", "", "OpenStreetMap contributors, CC-BY-SA", "http://tile.openstreetmap.fr/hot/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "19", "0"])
sources.append(["connections-xyz","OpenStreetMap Monochrome", "", "", "OpenStreetMap contributors, CC-BY-SA", "http://tiles.wmflabs.org/bw-mapnik/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "19", "0"])
sources.append(["connections-xyz","Strava All", "", "", "OpenStreetMap contributors, CC-BY-SA", "https://heatmap-external-b.strava.com/tiles/all/bluered/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "15", "0"])
sources.append(["connections-xyz","Strava Run", "", "", "OpenStreetMap contributors, CC-BY-SA", "https://heatmap-external-b.strava.com/tiles/run/bluered/%7Bz%7D/%7Bx%7D/%7By%7D.png?v=19", "", "15", "0"])
sources.append(["connections-xyz","Open Weather Map Temperature", "", "", "Map tiles by OpenWeatherMap, under CC BY-SA 4.0", "http://tile.openweathermap.org/map/temp_new/%7Bz%7D/%7Bx%7D/%7By%7D.png?APPID=1c3e4ef8e25596946ee1f3846b53218a", "", "19", "0"])
sources.append(["connections-xyz","Open Weather Map Clouds", "", "", "Map tiles by OpenWeatherMap, under CC BY-SA 4.0", "http://tile.openweathermap.org/map/clouds_new/%7Bz%7D/%7Bx%7D/%7By%7D.png?APPID=ef3c5137f6c31db50c4c6f1ce4e7e9dd", "", "19", "0"])
sources.append(["connections-xyz","Open Weather Map Wind Speed", "", "", "Map tiles by OpenWeatherMap, under CC BY-SA 4.0", "http://tile.openweathermap.org/map/wind_new/%7Bz%7D/%7Bx%7D/%7By%7D.png?APPID=f9d0069aa69438d52276ae25c1ee9893", "", "19", "0"])
sources.append(["connections-xyz","CartoDb Dark Matter", "", "", "Map tiles by CartoDB, under CC BY 3.0. Data by OpenStreetMap, under ODbL.", "http://basemaps.cartocdn.com/dark_all/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "20", "0"])
sources.append(["connections-xyz","CartoDb Positron", "", "", "Map tiles by CartoDB, under CC BY 3.0. Data by OpenStreetMap, under ODbL.", "http://basemaps.cartocdn.com/light_all/%7Bz%7D/%7Bx%7D/%7By%7D.png", "", "20", "0"])
sources.append(["connections-xyz","Bing VirtualEarth", "", "", "", "http://ecn.t3.tiles.virtualearth.net/tiles/a{q}.jpeg?g=1", "", "19", "1"])

```

# 4.  N：30°29′13.38″  E：114°28′49.83″

  Center (N , E)：30.487051085698166, 114.48050921768188  
  N：30°29′13.38″  E：114°28′49.83″

# 5. GraphicsContext
```
public class LayoutControl extends Control {
    private static final int DEFAULT_WIDTH = 512;
    private static final int DEFAULT_HEIGHT = 512;
    private LayoutTool layoutTool = null;
    private Layout mLayout = null;
    private ILayoutSizeChangeListener mSizeChageListener = this::onSizeChanged;
    private ILayoutSelectSetChangeListener mSelectSetChangeListener = this::onSelectSetChanged;
    //endregion

    //region 构造
    public LayoutControl() {

        zoomCanvas = new javafx.scene.canvas.Canvas(DEFAULT_WIDTH, DEFAULT_HEIGHT);
        this.getChildren().add(zoomCanvas);
        GraphicsContext gczoom = zoomCanvas.getGraphicsContext2D();
        gczoom.setFill(javafx.scene.paint.Color.rgb(200, 200, 200, 0.4));
        gczoom.setStroke(javafx.scene.paint.Color.rgb(100, 110, 110, 1));
        gczoom.setLineWidth(0.02);
        zoomCanvas.setVisible(false);
        zoomCanvas.heightProperty().bind(this.heightProperty());
        zoomCanvas.widthProperty().bind(this.widthProperty());

        this.setPrefWidth(DEFAULT_WIDTH);
        this.setPrefHeight(DEFAULT_HEIGHT);
        LayoutControlNative.jni_OnSize(layoutControlNative, 0, DEFAULT_WIDTH, DEFAULT_HEIGHT);
        this.widthProperty().addListener((observable, oldValue, newValue) -> {
            double height = this.getHeight();
            if (height > 0.001 && newValue.intValue() > 0) {
                if (isFirstShow) {
                    isFirstShow = false;
                    Rect devRc = new Rect();
                    devRc.setXMin(0.0);
                    devRc.setYMin(0.0);
                    devRc.setXMax(newValue.doubleValue());
                    devRc.setYMax(height);
                    setDeviceRect(devRc);
                    setClientRect(devRc);
                    restoreWnd();
                } else {
                    LayoutControlNative.jni_OnSize(layoutControlNative, 0, newValue.intValue(), (int) height);
                }
            }
        });
        this.heightProperty().addListener((observable, oldValue, newValue) -> {
            double width = this.getWidth();
            if (width > 0.001 && newValue.intValue() > 0) {
                if (isFirstShow) {
                    isFirstShow = false;
                    Rect devRc = new Rect();
                    devRc.setXMin(0.0);
                    devRc.setYMin(0.0);
                    devRc.setXMax(width);
                    devRc.setYMax(newValue.doubleValue());
                    getTransformation().setDeviceRect(devRc);
                    getTransformation().setClientRect(devRc);
                    restoreWnd();
                } else {
                    LayoutControlNative.jni_OnSize(layoutControlNative, 0, (int) width, newValue.intValue());
                }
            }
        });

        this.setFocusTraversable(true);
    }

	
    public LayoutTool getLayoutTool() {
        return this.layoutTool;
    }

    private InteractionListener getInteractionListener() {
        return this.interactionListener;
    }

    private void setInteractionListener(InteractionListener interactionListener) {
        //Check.throwIfNull(interactionListener, "interactionListener");
        if (this.interactionListener != null) {
            this.interactionListener.onRemoved();
        }

        this.interactionListener = interactionListener;
        if (this.interactionListener == null)
            return;

        this.setOnMousePressed(this.interactionListener::onMousePressed);
        this.setOnMouseClicked(this.interactionListener::onMouseClicked);
        this.setOnMouseDragged(this.interactionListener::onMouseDragged);
        this.setOnMouseReleased(this.interactionListener::onMouseReleased);
        this.addEventFilter(MouseEvent.MOUSE_MOVED, this.interactionListener::onMouseMoved);
        this.setOnKeyPressed(this.interactionListener::onKeyPressed);
        this.setOnKeyReleased(this.interactionListener::onKeyReleased);
        this.setOnScroll(this.interactionListener::onScroll);
        this.setOnScrollStarted(this.interactionListener::onScrollStarted);
        this.setOnScrollFinished(this.interactionListener::onScrollFinished);
        this.interactionListener.onAdded();
    }

	
    public void setLayout(Layout layout) {
        if (this.mLayout != null) {
            this.mLayout.removeSizeChagedListener(mSizeChageListener);
            this.mLayout.removeSelectSetChangedListener(mSelectSetChangeListener);
        }

        this.mLayout = layout;
        if (layout == null)
            return;
        LayoutControlNative.jni_SetLayout(layoutControlNative, this.mLayout.getHandle());

        this.mLayout.addSizeChagedListener(mSizeChageListener);
        this.mLayout.addSelectSetChangedListener(mSelectSetChangeListener);

        this.layoutTool = new LayoutTool(this);
        this.setInteractionListener(layoutTool);
    }


    protected boolean isPointHit(MouseEvent event) {

        int  offsetx = (int) Math.abs(event.getX() - this.mLastPressX);
        int  offsety = (int) Math.abs(event.getY() - this.mLastPressY);

        boolean isPntClick = (offsetx <= 5 || offsety <= 5);
        return isPntClick;
    }

    public void onRefreshSelectRegion(MouseEvent event) {

        boolean isPntClick = isPointHit(event);
        if(isPntClick){
            bvalidDrag = false;
            return;
        }

        Bounds b = this.layoutControl.getBoundsInLocal();
        com.butter.geometry.Rect devRect = new com.butter.geometry.Rect();
        devRect.setXMin(Math.min(event.getX(), this.mLastPressX));
        devRect.setXMax(Math.max(event.getX(), this.mLastPressX));
        devRect.setYMin(b.getHeight() - Math.max(event.getY(), this.mLastPressY));
        devRect.setYMax(b.getHeight() - Math.min(event.getY(), this.mLastPressY));

        try {
            GraphicsContext gcc = this.layoutControl.getZoomCanvas().getGraphicsContext2D();
            gcc.clearRect(0, 0, this.layoutControl.getZoomCanvas().getWidth(), this.layoutControl.getZoomCanvas().getHeight());
            gcc.setLineDashes(0, 0);
            gcc.strokeRect(Math.min(event.getX(), this.mLastPressX), Math.min(event.getY(), this.mLastPressY), Math.abs(event.getX() - this.mLastPressX), Math.abs(event.getY() - this.mLastPressY));
            this.layoutControl.getZoomCanvas().setVisible(true);
            bvalidDrag = true;

        }catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```

# 6. Interaction Listener
```

import javafx.scene.input.KeyEvent;
import javafx.scene.input.MouseEvent;
import javafx.scene.input.RotateEvent;
import javafx.scene.input.ScrollEvent;
import javafx.scene.input.SwipeEvent;
import javafx.scene.input.TouchEvent;
import javafx.scene.input.ZoomEvent;

public interface InteractionListener {
    default void onAdded() {
    }

    default void onRemoved() {
    }

    default void onMouseClicked(MouseEvent event) {
    }

    default void onMouseDragged(MouseEvent event) {
    }

    default void onMousePressed(MouseEvent event) {
    }

    default void onMouseReleased(MouseEvent event) {
    }

    default void onMouseMoved(MouseEvent event) {
    }

    default void onKeyPressed(KeyEvent event) {
    }

    default void onKeyReleased(KeyEvent event) {
    }

    default void onKeyTyped(KeyEvent event) {
    }

    default void onScroll(ScrollEvent event) {
    }

    default void onZoom(ZoomEvent event) {
    }

    default void onRotate(RotateEvent event) {
    }

    default void onMouseEntered(MouseEvent event) {
    }

    default void onMouseExited(MouseEvent event) {
    }

    default void onScrollStarted(ScrollEvent event) {
    }

    default void onScrollFinished(ScrollEvent event) {
    }

    default void onZoomStarted(ZoomEvent event) {
    }

    default void onZoomFinished(ZoomEvent event) {
    }

    default void onRotationStarted(RotateEvent event) {
    }

    default void onRotationFinished(RotateEvent event) {
    }
}
```

# 7. Callback


```
class ControlListener : public GListener
{
private:
	JavaVM* m_jvm;
	jobject m_jobj;
public:
	ControlListener(JavaVM* env, jobject obj)
	{
		m_jvm = env;
		m_jobj = obj;
	}
	virtual void OnFire(::EventType type, GEventArgs *args)
	{

		switch (type)
		{
		case MapViewAfterReFresh:
		{
			CGViewAfterReFreshArgs* refreshArgs = (CGViewAfterReFreshArgs*)args;
			JNIEnv* jniEnv = NULL;
			int state = m_jvm->GetEnv((void**)&jniEnv, 0x00010008);// JNI_VERSION_1_8);
			bool isNeedDetach = false;
			if (state != JNI_OK) {
				isNeedDetach = true;
#ifndef LINUX_ANDROID
				m_jvm->AttachCurrentThread((void**)&jniEnv, NULL);
#else
				m_jvm->AttachCurrentThread((JNIEnv**)&jniEnv, NULL);
#endif
			}
			CSmartFile* sFile = refreshArgs->GetImageBuffer();
			unsigned long dataLen = sFile->GetLength();
			jbyteArray data = jniEnv->NewByteArray(dataLen);

			char* ptBuf = new char[dataLen];

			sFile->SeekToBegin();
			sFile->Read(ptBuf, dataLen);
			jniEnv->SetByteArrayRegion(data, 0, dataLen, (jbyte*)ptBuf);
			safe_delete_array(ptBuf);

			jclass cls = jniEnv->GetObjectClass(m_jobj);
			jmethodID mid = jniEnv->GetMethodID(cls, "onCallbackAfterReFresh", "([B)V");
			jniEnv->CallVoidMethod(m_jobj, mid, data);
			jniEnv->DeleteLocalRef(data);
			jniEnv->DeleteLocalRef(cls);

			if (isNeedDetach)
				m_jvm->DetachCurrentThread();
		}
		break;
		case OverlayAfterRefresh:
		{
			CGViewAfterReFreshArgs* refreshArgs = (CGViewAfterReFreshArgs*)args;
			JNIEnv* jniEnv = NULL;
			int state = m_jvm->GetEnv((void**)&jniEnv, 0x00010008);// JNI_VERSION_1_8);
			bool isNeedDetach = false;
			if (state != JNI_OK) {
				isNeedDetach = true;
#ifndef LINUX_ANDROID
				m_jvm->AttachCurrentThread((void**)&jniEnv, NULL);
#else
				m_jvm->AttachCurrentThread((JNIEnv**)&jniEnv, NULL);
#endif
			}
			CSmartFile* sFile = refreshArgs->GetImageBuffer();
			unsigned long dataLen = sFile->GetLength();
			jbyteArray data = jniEnv->NewByteArray(dataLen);

			char* ptBuf = new char[dataLen];

			sFile->SeekToBegin();
			sFile->Read(ptBuf, dataLen);
			jniEnv->SetByteArrayRegion(data, 0, dataLen, (jbyte*)ptBuf);
			safe_delete_array(ptBuf);

			jclass cls = jniEnv->GetObjectClass(m_jobj);
			jmethodID mid = jniEnv->GetMethodID(cls, "onCallbackAfterOverlayReFresh", "([B)V");
			jniEnv->CallVoidMethod(m_jobj, mid, data);
			jniEnv->DeleteLocalRef(data);
			jniEnv->DeleteLocalRef(cls);

			if (isNeedDetach)
				m_jvm->DetachCurrentThread();
		}
		break;
		case dataViewBeginDrawing:
		{
			JNIEnv* jniEnv = NULL;
			int state = m_jvm->GetEnv((void**)&jniEnv, 0x00010008);// JNI_VERSION_1_8);
			bool isNeedDetach = false;
			if (state != JNI_OK) {
				isNeedDetach = true;
#ifndef LINUX_ANDROID
				m_jvm->AttachCurrentThread((void**)&jniEnv, NULL);
#else
				m_jvm->AttachCurrentThread((JNIEnv**)&jniEnv, NULL);
#endif
			}

			jclass cls = jniEnv->GetObjectClass(m_jobj);
			jmethodID mid = jniEnv->GetMethodID(cls, "onCallbackBeforeRefesh", "()V");
			jniEnv->CallVoidMethod(m_jobj, mid);
			jniEnv->DeleteLocalRef(cls);

			if (isNeedDetach)
				m_jvm->DetachCurrentThread();
		}
		break;
		default:
			break;
		}
	}
};
class CGMapViewX : public CGMapView
{
private:
	ControlListener* m_listenner;
	jobject m_job;
	JavaVM* m_jvm;
public:
	CGMapViewX(JNIEnv* env, jobject obj) :m_listenner(NULL)
	{
		env->GetJavaVM(&m_jvm);
		m_job = env->NewGlobalRef(obj);

		m_listenner = new ControlListener(m_jvm, m_job);
		this->Add(m_listenner, allEvent);
		this->SetImageBufferType(1);//bgra
	}
	virtual ~CGMapViewX()
	{
		delete m_listenner;

		JNIEnv* jniEnv = NULL;
		int state = m_jvm->GetEnv((void**)&jniEnv, 0x00010008);// JNI_VERSION_1_8);
		bool isNeedDetach = false;
		if (state != JNI_OK) {
			isNeedDetach = true;
#ifndef LINUX_ANDROID
			m_jvm->AttachCurrentThread((void**)&jniEnv, NULL);
#else
			m_jvm->AttachCurrentThread((JNIEnv**)&jniEnv, NULL);
#endif
		}
		jniEnv->DeleteGlobalRef(m_job);

		if (isNeedDetach)
			m_jvm->DetachCurrentThread();
	}
};

JNIEXPORT jlong JNICALL Java_com_butter_controls_ControlNative_jni_1CreateObj(JNIEnv *env, jclass jclassobj, jobject dataobj)
{
	CGMapViewX* pMapView = new CGMapViewX(env, dataobj);
	return (jlong)pMapView;
}
JNIEXPORT void JNICALL Java_com_butter_controls_ControlNative_jni_1DeleteObj(JNIEnv *env, jclass jclassobj, jlong handle)
{
	CGMapViewX* pMapView = (CGMapViewX*)handle;
	safe_delete(pMapView);
}


```
# 8. A dynamic link library (DLL) initialization routine failed

![initialization routine failed](./img/8_initialization_routine_failed.png)

```
use IntelliJ IDEA to debug some program, when load some dynamic link library (*.dll),
throw out some error infomation: "A dynamic link library (DLL) initialization routine failed" ;

when check the dll file ,the file exist and the located path has been set in the Computer PATH ENV,
besides this,use the dependency tool(dependency walker), we get the full complete dependent file list in the table view;

the method to solve this situation:

1. cmd --->regedit 
2. locate the target program node;
   eg: HKEY_CURRENT_USER\Software\MapCycle\AppProduct\Developer
3. delete the java node which set the java infomation,
   and node name make consist of the HEX code;


```
 regedit path
![HKEY_CURRENT_USER_Software_MapCycle](./img/8_initialization_routine_failed_regedit.png)



# 9. Debug cpp module in Android studio

![Debug cpp module in Android studio](./img/9_debug_cpp_in_androidstudio.png)
```
STEPS:

1. open project in android studio;
2. select Android Node,not Project Node,
3. select app tree root node,and click the mouse right button,
   get the menu lists;
4. click 'Link c++ project with Gradle' from the menu
   the show the configure window
5. if the *.mk file is Ok,choose ndk-build for Build system;
   then choose the file 'Android.mk' in the project path;
6.press ok,add the mk config into the build.gradle;

	android {
		compileSdkVersion 26
		defaultConfig {
			applicationId "com.map.cycle"
			minSdkVersion 21
			targetSdkVersion 26
			versionCode 1
			versionName "1.0"
			testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
			multiDexEnabled true
			ndk {
				abiFilters "armeabi-v7a"
			}
		}
		externalNativeBuild {
			ndkBuild {
				path file('../../code/build/map/jni/Android.mk')
			}
		}
	}
```



# 10. signal SIGBUS: illegal alignment

For some architecture,eg. SPARC、m68k, when access the illegal alignment address,the CPU will throw the SIGBUS signal;
but for some other architecture,eg. IA-32 from Intel,that is OK;

the last code for the legal alignment address usually is '0','4','8','C';

## 10.1 WHAT is ALIGNMENT


![10_signal_SIGBUS_illegal_alignment_align](./img/10_signal_SIGBUS_illegal_alignment_align.png)

## 10.2 METHOD

```
1. use memcpy ,not use this illegal alignment address directly

2. redesign the data structure with the reserved space to make the address alignment legally;
   

```

![10_signal_SIGBUS_illegal_alignment_method](./img/10_signal_SIGBUS_illegal_alignment_method.png)


## 10.3 DEMO

```
testDemo:

	bool Align_noTemplate(float& t, float f){
		t = f; return true;
	} 
	template<> inline bool Align(float& t, float f){
		t = f; return true;
	}
	template<class T> bool Align_Float(T& t, T f){
		t = f; return true;
	}
	test()
	{
	LIN_INFO linInfo;float fVal = 1.8;
	linInfo.xscale = fVal;                        ///1.use =,          OK
	memcpy(&linInfo.xscale, &fVal, sizeof(float));///2.memcpy,         OK
	Align_noTemplate(linInfo.xscale, fVal );      ///3.no template     OK
	Align_Float(linInfo.xscale, fVal );           ///4.template.temp   OK
	linInfo.xscale = fVal;
	Align_Float(linInfo.xscale, fVal );           ///5.template.direct BAD
	}


	char szBuffer[512] = { 0 };
	memset(szBuffer,1,sizeof(char)*512);
	for(int m = 0; m < 16;m++){
		float fVal = 1.8;
		char* ptr = (szBuffer+m);
		float* pfVal = (float*)ptr;
		fVal = *pfVal; 
		Align_noTemplate(*pfVal, fVal); ///1. OK
		Align_Float(*pfVal, fVal);      ///2. sometimes BAD
	}


```

## 10.4 Link ERROR

![initialization routine failed](./img/10_signal_SIGBUS_illegal_alignment.png)

```
Case description:

1. the application is broken down and throw the SIGBUS signal; 
   the signal give the illegal allignment detail;

2. when we check the logic,for the variable which has the illegal allignment address,
   we have fix it with 'memcpy' method; 
   despite the fixes,the cpu give the SIGBUS signal;

3. then debug the function,in the debuger stack we find the error line is 'Gray' ,
   but the upper logic function line is 'Black'; the Gray line is not located in the right place;
   To check this,we add the log with 'LOGITEST' in the target function,recompile and run,
   we CANNOT get the log in the logcat window; which indicate that maybe the library link the the wrong function;

4. when rename and relink the target function ,compile and run, we can get the log and the SIGBUS signal disappear;

5. for this case, from the outside,the illegal allignment SIGBUS is the orignal question essentially;  
   deal it with 'memcpy' method is the right choice;
   But in the library we maybe find the target function more than once which may have different logic,
   especially when we merge many many static libs into one dynamic library;

6. rename the target function or recall the function with the unique namespace ,
   through this we fix this question;

7. so when we copy the existed functions,prefer to Surround the functions with the unique name space or rename the functions ,
   to make them unambiguous;
```


# 11. token

```
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01
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01
```
# 12. OGCWMTS

```
1. OGC WMTS (Web MAP Tile Service) is a standard for web map; 
   but for different map vendor,they may use it vary from each other,especially for the non-mainstream vendor;
2. for mainstream vendor,they list the readME document in the easy access website usually; 
   Follow the the readMe document,we can use the tile service easily with not that much time and effort;

   but for no-mainstream vendor,How can we do?

3.for example ,we get the server instance url:
  https://fxpc.mem.gov.cn/data_preparation/a12eadf6-1a57-43fe-9054-2e22277bd553/8c6a43c6-fa48-4f2b-9296-f2c02c4474a7/img08/wmts/2CA54B6D242305ABF3822EC38D121CD9/WMTSCapabilities.xml 

  when we access the url above in chrome explore,we get some information about the server capabilities information,like this:


<ows:ServiceIdentification>
...
</ows:ServiceIdentification>
<ows:ServiceProvider>
...
</ows:ServiceProvider>
<ows:OperationsMetadata>
...
</ows:OperationsMetadata>
<Contents>
<Layer>
<Format>tiles</Format>
<TileMatrixSetLink>
<TileMatrixSet>default028mm</TileMatrixSet>
</TileMatrixSetLink>
<TileMatrixSetLink>
<TileMatrixSet>nativeTileMatrixSet</TileMatrixSet>
</TileMatrixSetLink>
</Layer>
<TileMatrixSet>
...
</TileMatrixSet>
<TileMatrixSet>
...
</TileMatrixSet>
</Contents>
</Capabilities>


the server url parse ENGINE parse the the above url  to url:

https://fxpc.mem.gov.cn/data_preparation/a12eadf6-1a57-43fe-9054-2e22277bd553/8c6a43c6-fa48-4f2b-9296-f2c02c4474a7/img08/wmts?service=WMTS&version=1.0.0&request=GetCapabilities

OR

https://fxpc.mem.gov.cn/data_preparation/a12eadf6-1a57-43fe-9054-2e22277bd553/8c6a43c6-fa48-4f2b-9296-f2c02c4474a7/wmts?layer=img08&service=WMTS&version=1.0.0&request=GetCapabilities

so we get the capabilities information;

then we want to get the tile image buffer stream, HOW?

4. for Tianditu WMTS,we get the instruction from '  http://lbs.tianditu.gov.cn/server/MapService.html ' ,
the example url is :
http://t0.tianditu.gov.cn/img_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILEMATRIX={z}&TILEROW={x}&TILECOL={y}&tk=yourtoken

According the url above,we make the target url:

 https://fxpc.mem.gov.cn/data_preparation/a12eadf6-1a57-43fe-9054-2e22277bd553/8c6a43c6-fa48-4f2b-9296-f2c02c4474a7/img08/wmts/?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img08&STYLE=default&FORMAT=tiles&TILEMATRIXSET=nativeTileMatrixSet&TILEMATRIX=1&TILEROW=0&TILECOL=1

 or

 https://fxpc.mem.gov.cn/data_preparation/a12eadf6-1a57-43fe-9054-2e22277bd553/8c6a43c6-fa48-4f2b-9296-f2c02c4474a7/img08/wmts/2CA54B6D242305ABF3822EC38D121CD9?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img08&STYLE=default&FORMAT=tiles&TILEMATRIXSET=nativeTileMatrixSet&TILEMATRIX=1&TILEROW=0&TILECOL=1

 access them in chrome explore, Both ERROR;

 5. add the token? no token;
   access the original url in Arcgis pro 2.5;we get the tile stream and display normally;
   WHY? perhaps the wrong target url;
   How to get the right url;
6. use fiddler  (https://www.telerik.com/fiddler);
   run fiddler,at the same time,refresh the target layer in Arcgis pro 2.5;
   the fiddler get some useful information,in fiddler,pick the target line;
   get :

   data_preparation/a12eadf6-1a57-43fe-9054-2e22277bd553/8c6a43c6-fa48-4f2b-9296-f2c02c4474a7/wmts/2CA54B6D242305ABF3822EC38D121CD9?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img08&STYLE=default&FORMAT=tiles&TILEMATRIXSET=nativeTileMatrixSet&TILEMATRIX=4&TILEROW=2&TILECOL=12

   append host site https://fxpc.mem.gov.cn/ ;

   https://fxpc.mem.gov.cn/data_preparation/a12eadf6-1a57-43fe-9054-2e22277bd553/8c6a43c6-fa48-4f2b-9296-f2c02c4474a7/wmts/2CA54B6D242305ABF3822EC38D121CD9?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=img08&STYLE=default&FORMAT=tiles&TILEMATRIXSET=nativeTileMatrixSet&TILEMATRIX=4&TILEROW=2&TILECOL=12

   access the above url in chrome explore,OK, we get it,we get the tile image stream;
   




```


# 13. View dynamic library dependency

   * for Windows,  use dependency-walker
     
   * for linux,  use ldd command 
    ```
       ldd libjpeg.so
       ldd: warning: you do not have execution permission for `./libjpeg.so'
	   linux-vdso.so.1 =>  (0x00007ffea15a2000)
	   libc.so.6 => /lib64/libc.so.6 (0x00007f8949b12000)
	   /lib64/ld-linux-x86-64.so.2 (0x00007f894a138000)
    ```
   * for android , use readelf in ndk,

	```
	PS D:\android\program\adt\ndk_r16b\toolchains\arm-linux-androideabi-4.9\prebuilt\windows-x86_64\arm-linux-androideabi\bin> 
	 .\readelf.exe -d D:\android\samples\app\libs\armeabi-v7a\libQt5Core.so

	Dynamic section at offset 0x72f0a4 contains 33 entries:
	Tag        Type                         Name/Value
	0x00000003 (PLTGOT)                     0x730784
	0x00000002 (PLTRELSZ)                   28896 (bytes)
	0x00000017 (JMPREL)                     0x73c8c
	0x00000014 (PLTREL)                     REL
	0x00000011 (REL)                        0x6aba4
	0x00000012 (RELSZ)                      37096 (bytes)
	0x00000013 (RELENT)                     8 (bytes)
	0x6ffffffa (RELCOUNT)                   1956
	0x00000006 (SYMTAB)                     0x1f0
	0x0000000b (SYMENT)                     16 (bytes)
	0x00000005 (STRTAB)                     0x18b50
	0x0000000a (STRSZ)                      236876 (bytes)
	0x6ffffef5 (GNU_HASH)                   0x5289c
	0x00000004 (HASH)                       0x5d790
	0x00000001 (NEEDED)                     Shared library: [libz.so]
	0x00000001 (NEEDED)                     Shared library: [libc++_shared.so]
	0x00000001 (NEEDED)                     Shared library: [liblog.so]
	0x00000001 (NEEDED)                     Shared library: [libm.so]
	0x00000001 (NEEDED)                     Shared library: [libdl.so]
	0x00000001 (NEEDED)                     Shared library: [libc.so]
	0x0000000e (SONAME)                     Library soname: [libQt5Core.so]
	0x0000001a (FINI_ARRAY)                 0x730088
	0x0000001c (FINI_ARRAYSZ)               16 (bytes)
	0x00000019 (INIT_ARRAY)                 0x730098
	0x0000001b (INIT_ARRAYSZ)               12 (bytes)
	0x0000001e (FLAGS)                      BIND_NOW
	0x6ffffffb (FLAGS_1)                    Flags: NOW
	0x6ffffff0 (VERSYM)                     0x679fc
	0x6ffffffc (VERDEF)                     0x6ab28
	0x6ffffffd (VERDEFNUM)                  1
	0x6ffffffe (VERNEED)                    0x6ab44
	0x6fffffff (VERNEEDNUM)                 3
	0x00000000 (NULL)                       0x0
	```


# 14. Makefile error make (e=2): The system cannot find the file specified


1. when compile Qt for android in Windows Environment, crash the following case:
   ```
   cannot find E:\Qt\qtbase\src\android\jar\QtAndroid.jar
   jar cf QtAndroid.jar -C .classes .
   process_begin: CreateProcess(NULL, jar cf QtAndroid.jar -C .classes ., ...) failed.
   make (e=2): The system cannot find the file specified
   ```
2. almost certainly complaining that Windows cannot find 'jar'.
   
   This is almost certainly because the value of %PATH% (or whatever) is different when make spawns a shell/console then when you have it open manually.
   
   Compare the values to confirm that. Then either use the full path to 'jar' in the makefile recipe or ensure that the value of PATH is set correctly for make's usage

3. in console , print 'where java',then show the full path of java.exe;
   but when print 'where jar',cannot show the full path of 'jar.exe';
   in path %JAVA_HOME%\bin, we can find jar.exe  essentially, but now we cannot;

   when we check the JAVA folder in other PC ,we find that file 'jar.exe';
   then copy the qt source code, compile java code,
   run 'jar cf QtAndroid.jar -C .classes .';
   that work OK;

   so  the java environment maybe has been broken sometimes when we install other program;

   copy the jar.exe into the target PC or reinstall JDK in the target PC,
   then check,OK,we solve this problem;

4. besides this,when search this topic key words,we find the similar question;
   almost certainly complaining that Windows cannot find some target process,

   what we should do is locate the target process path,make the compiler know the path,
   find the target process in the path;

5. fro more information, please visit 
   * https://stackoverflow.com/questions/33674973/makefile-error-make-e-2-the-system-cannot-find-the-file-specified  
   or
   * https://stackoverflow.com/questions/7291442/error-cannot-run-program-jar-createprocess-error-2-the-system-cannot-find-t

# 15. compile Qt for android in Windows Environment

* for more infomation,please visit https://doc.qt.io/qt-5/android-building.html 

## 15.1. Prepare

To build Qt for Android under a Windows environment, follow the steps below:

Preparing the Build Environment
Install the following:

* A JDK package such as AdoptOpenJDK, JDK, or OpenJDK.
* MinGW 7.3 toolchain
* Perl

Then set the respective environment variables from the Environment Variables system UI, or from the build command line prompt. For the default Command prompt:

* set JDK_ROOT=<JDK_ROOT_PATH>\bin
* set MINGW_ROOT=<MINGW_ROOT_PATH>\bin
* set PERL_ROOT=<PERL_ROOT_PATH>\bin
* set PATH=%MINGW_ROOT%;%PERL_ROOT%;%JDK_ROOT%:%PATH%

Then, in the command line prompt, verify that:

```
where gcc.exe
The command should list gcc.exe under the path <MINGW_ROOT> first.

where mingw32-make.exe
The command should list mingw32-make.exe under the path <MINGW_ROOT> first.

where javac.exe
The command should list javac.exe under the path <JDK_ROOT> first.

```
* Note: JDK 11 or earlier must be used to properly build Qt for Android.

* Note: Qt for Android does not support building with Microsoft Visual C++ (MSVC), we only support building with MinGW.

* If you have downloaded the source code archive from Qt Downloads, then unpack the archive if you have not done so already.  qt-everywhere-src-%VERSION%.tar.xz package

   
## 15.2. Compile


1. save the following compile script into file *.bat
2. open cmd console,open the source code path,
3. run the *.bat created in step 1;
   
### compile script
   
```
:::::::::::::::::::::::::::::::::::::compile script ::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::arm64-v8a  armeabi-v7a  armeabi::::::::::::::::::::

@echo %PATH%

set "ANDROID_API_VERSION=android-21"
set "ANDROID_SDK_ROOT=D:\android\program\adt\sdk"
set "ANDROID_TARGET_ARCH=arm64-v8a"
set "ANDROID_BUILD_TOOLS_REVISION=21.1.2"
set "ANDROID_NDK_PATH=D:\android\program\adt\ndk-r20b"
set "ANDROID_TOOLCHAIN_VERSION=4.9"
set "ANDROID_NDK_HOST=windows-x86_64"
set "BUILD_PATH=E:\Qt\build"
set "PREFIX_PATH=E:\Qt\install"

set JDK_ROOT=D:\Pragram\Java\jdk1.8.0_162\bin
set MINGW_ROOT=E:\SoftWare\mingw32\bin
set PERL_ROOT=C:\Strawberry\perl\bin
set PATH=%MINGW_ROOT%;%PERL_ROOT%;%JDK_ROOT%:%PATH%

::configure.bat -prefix F:\Qt\build -platform win32-g++ -opengl es2 -xplatform android-g++ -android-ndk %ANDROID_NDK_PATH% -android-sdk %ANDROID_SDK_ROOT% -android-toolchain-version 4.9 -nomake tests -nomake examples
::configure.bat  -developer-build -platform win32-g++ -opengl dynamic -xplatform android-g++ -android-ndk %ANDROID_NDK_PATH% -android-sdk %ANDROID_SDK_ROOT% -android-toolchain-version 4.9 -nomake tests -nomake examples

::configure.bat -platform win32-g++ -opengl es2 -xplatform android-g++ -android-ndk %ANDROID_NDK_PATH% -android-sdk %ANDROID_SDK_ROOT% -android-toolchain-version 4.9 -android-ndk-host windows-x86_64 -android-ndk-platform android-21 -opensource -confirm-license -nomake tests -nomake examples -static
::configure.bat -opengl dynamic -no-opengl

pause

mkdir %BUILD_PATH%
mkdir %PREFIX_PATH%
mkdir %BUILD_PATH%\%ANDROID_TARGET_ARCH%
mkdir %PREFIX_PATH%\%ANDROID_TARGET_ARCH%
cd %BUILD_PATH%\%ANDROID_TARGET_ARCH%

..\..\5.12.5\configure.bat -platform win32-g++ -opengl es2 -xplatform android-clang -android-ndk %ANDROID_NDK_PATH% -android-sdk %ANDROID_SDK_ROOT% -android-toolchain-version 4.9 -android-ndk-host windows-x86_64 -android-ndk-platform android-21 -prefix %PREFIX_PATH%\%ANDROID_TARGET_ARCH% -opensource -confirm-license -nomake tests -nomake examples -shared

::mingw32-make -j4
mingw32-make

pause
:::::::::::::::::::::::::::::::::::::compile script::::::::::::::::::::::::::::::::::::
```
## 15.3 some problems

### Android linker: undefined reference to bsd_signal

* /android/ndk/platforms/android-9/arch-arm/usr/include/signal.h:113: error: undefined reference to 'bsd_signal'
  
Till android-19 inclusive NDK-s signal.h declared bsd_signal extern and signal was an inline calling bsd_signal.

Starting with android-21 signal is an extern and bsd_signal is not declared at all.

What's interesting, bsd_signal was still available as a symbol in NDK r10e android-21 libc.so (so there were no linking errors if using r10e), but is not available in NDK r11 and up.

Removing of bsd_signal from NDK-s android-21+ libc.so results in linking errors if code built with android-21+ is linked with static libs built with lower NDK levels that call signal or bsd_signal. Most popular library which calls signal is OpenSSL.

WARNING: Building those static libs with android-21+ (which would put signal symbol directly) would link fine, but would result in *.so failing to load on older Android OS devices due to signal symbol not found in theirs libc.so.

Therefore it's better to stick with <=android-19 for any code that calls signal or bsd_signal.

To link a library built with android-21 I ended up declaring a bsd_signal wrapper which would call bsd_signal from libc.so (it's still available in device's libc.so, even up to Android 7.0).

```

#if (__ANDROID_API__ > 19)
#include <android/api-level.h>
#include <android/log.h>
#include <signal.h>
#include <dlfcn.h>

extern "C" {
  typedef __sighandler_t (*bsd_signal_func_t)(int, __sighandler_t);
  bsd_signal_func_t bsd_signal_func = NULL;

  __sighandler_t bsd_signal(int s, __sighandler_t f) {
    if (bsd_signal_func == NULL) {
      // For now (up to Android 7.0) this is always available 
      bsd_signal_func = (bsd_signal_func_t) dlsym(RTLD_DEFAULT, "bsd_signal");

      if (bsd_signal_func == NULL) {
        // You may try dlsym(RTLD_DEFAULT, "signal") or dlsym(RTLD_NEXT, "signal") here
        // Make sure you add a comment here in StackOverflow
        // if you find a device that doesn't have "bsd_signal" in its libc.so!!!

        __android_log_assert("", "bsd_signal_wrapper", "bsd_signal symbol not found!");
      }
    }

    return bsd_signal_func(s, f);
  }
}
#endif

```
PS. Looks like the bsd_signal symbol will be brought back to libc.so in NDK r13:
for more information ,please visit 
 * https://stackoverflow.com/questions/36746904/android-linker-undefined-reference-to-bsd-signal
   
 * https://github.com/android-ndk/ndk/issues/160#issuecomment-236295994



# 16. link static c++  'libc++_static.a' when compile shared dynamic Qt

when compile dynamic Qt library,we use the configure item -shared;
then qmake generate *.pro file into the makefile whith the corresponding configuration;
in the end ,we get the *.so file which link the dynamic c++  file "libc++_shared.so";

```
PS D:\android\program\adt\ndk_r16b\toolchains\arm-linux-androideabi-4.9\prebuilt\windows-x86_64\arm-linux-androideabi\bin> 
.\readelf.exe -d E:\qt\libQt5Core.so

Dynamic section at offset 0x609670 contains 31 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [libz.so]
 0x0000000000000001 (NEEDED)             Shared library: [libc++_shared.so]
 0x0000000000000001 (NEEDED)             Shared library: [liblog.so]
 0x0000000000000001 (NEEDED)             Shared library: [libm.so]
 0x0000000000000001 (NEEDED)             Shared library: [libdl.so]
 0x0000000000000001 (NEEDED)             Shared library: [libc.so]
 0x000000000000000e (SONAME)             Library soname: [libQt5Core.so]

```

## How to link the static C++?

1. locate the makefile which match with 'corelib.pro';
2. open the makefile,find the TAG "LIBS",from this tag,we know the libraries which libQt*.so depend on ,inculde libc++_shared.so ;
  
	```
	LIBS          = $(SUBLIBS)  D:/android/program/adt/ndk-r20b/platforms/android-21/arch-arm/usr/lib/libz.so 
	E:/Qt/buildr20b/qtbase/lib/libqtpcre2.a 
	-LD:\android\program\adt\ndk-r20b/sources/cxx-stl/llvm-libc++/libs/armeabi-v7a D:\android\program\adt\ndk-r20b/sources/cxx-stl/llvm-libc++/libs/armeabi-v7a/libc++.so.21 -llog -lz -lm -ldl -lc 
	-LD:\android\program\adt\ndk-r20b/sources/cxx-stl/llvm-libc++/libs/armeabi-v7a D:\android\program\adt\ndk-r20b/sources/cxx-stl/llvm-libc++/libs/armeabi-v7a/libc++.so.21 -llog -lz -lm -ldl -lc

	```

3. in source code folder ../qtbase/ , search key words '-llog -lz -lm -ldl -lc' ;
	we find the file 'android-base-tail.conf'
	```
	QMAKE_LIBS_PRIVATE      = $$ANDROID_CXX_STL_LIBS -llog -lz -lm -ldl -lc
	```
4. then search key words 'ANDROID_CXX_STL_LIBS'
   we find the file 'qmake.conf'

	```
	exists($$ANDROID_SOURCES_CXX_STL_LIBDIR/libc++.so): \
		ANDROID_CXX_STL_LIBS = -lc++
	else: \
		ANDROID_CXX_STL_LIBS = $$ANDROID_SOURCES_CXX_STL_LIBDIR/libc++.so.$$replace(ANDROID_PLATFORM, "android-", "")

	```
5. according to the tag 'LIBS' information list in step 2,
  guess 'exists($$ANDROID_SOURCES_CXX_STL_LIBDIR/libc++.so)' return false; so modify file 'qmake.conf':

	```
	exists($$ANDROID_SOURCES_CXX_STL_LIBDIR/libc++.so): \
		ANDROID_CXX_STL_LIBS = -lc++
	else: \
		ANDROID_CXX_STL_LIBS = $$ANDROID_SOURCES_CXX_STL_LIBDIR/libc++.a.$$replace(ANDROID_PLATFORM, "android-", "")
	```
6. rebuild Qt and check the file libQt*.so
  
	```
	PS D:\android\program\adt\ndk_r16b\toolchains\arm-linux-androideabi-4.9\prebuilt\windows-x86_64\arm-linux-androideabi\bin> 
	.\readelf.exe -d E:\Qt\libQt5Core.so

	Dynamic section at offset 0x440508 contains 32 entries:
	Tag        Type                         Name/Value
	0x00000003 (PLTGOT)                     0x441e74
	0x00000002 (PLTRELSZ)                   33536 (bytes)
	0x00000017 (JMPREL)                     0xa36a0
	0x00000014 (PLTREL)                     REL
	0x00000011 (REL)                        0x94eb8
	0x00000012 (RELSZ)                      59368 (bytes)
	0x00000013 (RELENT)                     8 (bytes)
	0x6ffffffa (RELCOUNT)                   2995
	0x00000006 (SYMTAB)                     0x1f0
	0x0000000b (SYMENT)                     16 (bytes)
	0x00000005 (STRTAB)                     0x20890
	0x0000000a (STRSZ)                      337204 (bytes)
	0x6ffffef5 (GNU_HASH)                   0x72dc4
	0x00000004 (HASH)                       0x80b74
	0x00000001 (NEEDED)                     Shared library: [libz.so]
	0x00000001 (NEEDED)                     Shared library: [liblog.so]
	0x00000001 (NEEDED)                     Shared library: [libm.so]
	0x00000001 (NEEDED)                     Shared library: [libdl.so]
	0x00000001 (NEEDED)                     Shared library: [libc.so]
	0x0000000e (SONAME)                     Library soname: [libQt5Core.so]
	0x0000001a (FINI_ARRAY)                 0x437228

	```  
  
   
# 17. rewind anticlockwise polypolygon


``` 
long ClockSpace::CheckPolyPolyGon(DOT* pDotBuf, long* ne, int na)
{
	if (na <= 0) return 0;
	else if (na == 1) return 1;

	long nDotIndex = 0; bool bClockwiseBase = false;
	for (int i = 0; i < na; ++i)
	{
		bool bClockwise = ClockSpace::IsClockwise(pDotBuf + nDotIndex, ne[i]);
		
		if (i == 0) 
			bClockwiseBase = bClockwise;
		else if (bClockwise == bClockwiseBase)
			ClockSpace::Rewind(pDotBuf + nDotIndex, ne[i]);
		nDotIndex += ne[i];
	}

	return na;
}

void   ClockSpace::Rewind(DOT *tp, long n)
{
	long i, mid;
	DOT tmp;
	mid = n / 2;
	for (i = 0; i < mid; i++)
	{
		tmp = tp[i];
		tp[i] = tp[n - i - 1];
		tp[n - i - 1] = tmp;
	}
}

 bool ClockSpace::IsClockwise(DOT* pDotBuf, long ne) {
	double maxY = 0.0;
	int index = 0;

	//找到Y值最大的点及其前一点和后一点
	for (int i = 0; i < ne; i++) {
		DOT* pt = pDotBuf+i;
		if (i == 0) {
			maxY = pt->y;
			index = 0;
			continue;
		}
		if (maxY < pt->y) {
			maxY = pt->y;
			index = i;
		}
	}

	int front = index == 0 ? ne - 2 : index - 1;
	int middle = index;
	int after = index + 1;

	//利用矢量叉积判断是逆时针还是顺时针。设矢量P = ( x1, y1 )，Q = ( x2, y2 )，则矢量叉积定义为由(0,0)、p1、p2和p1+p2    
	//所组成的平行四边形的带符号的面积，即：P × Q = x1*y2 - x2*y1，其结果是一个标量。
	//显然有性质 P × Q = - ( Q × P ) 和 P × ( - Q ) = - ( P × Q )。
	//叉积的一个非常重要性质是可以通过它的符号判断两矢量相互之间的顺逆时针关系：
	//若 P × Q > 0 , 则P在Q的顺时针方向。
	//若 P × Q < 0 , 则P在Q的逆时针方向。
	//若 P × Q = 0 , 则P与Q共线，但可能同向也可能反向。
	//解释：
	//a×b=(ay * bz - by * az, az * bx - ax * bz, ax * by - ay * bx) 又因为az bz都为0，所以a×b=（0，0， ax * by - ay * bx）
	//根据右手系（叉乘满足右手系），若 P × Q > 0,ax * by - ay * bx>0,也就是大拇指指向朝上，所以P在Q的顺时针方向，一下同理。

	DOT* frontPt = (pDotBuf + front); 
	DOT* middlePt = (pDotBuf + middle); 
	DOT* afterPt =  (pDotBuf + after);

	return (afterPt->x - frontPt->x) * (middlePt->y - frontPt->y)
		- (middlePt->x - frontPt->x) * (afterPt->y - frontPt->y) > 0.0;
}

```

# 18. Parse Segments in string

```
woid testDemo()
{
	std::vector<std::string>  values;
	std::string strInfo = "id;name;property";
	ParseSegments(strInfo, ';', values);
}
//////////////////////////////////////////
int ParseSegments(std::string strValue, char chPartition, std::vector<std::string> & values)
{
	size_t idx = 0;
	values.clear();
	std::vector<size_t> indexs;
	for (; idx < strValue.size(); idx++)
	{
		if (strValue[idx] == chPartition)
		{
			indexs.push_back(idx);
		}
		else if (idx == strValue.size() - 1)
		{
			indexs.push_back(idx + 1);
		}
	}
	size_t nPosBegin = 0;
	for (idx = 0; idx < indexs.size(); idx++)
	{
		values.push_back(strValue.substr(nPosBegin, indexs[idx] - nPosBegin));
		nPosBegin = indexs[idx] + 1;
	}
	return idx;
}


int  Replace(std::string& strTarget, char src, char tar)
{
	size_t nPos = strTarget.npos; size_t nCount = 0;
	while ((nPos = strTarget.find(src)) != strTarget.npos)
	{
		strTarget.replace(nPos, sizeof(char), sizeof(char), tar);
		nCount++;
	}
	return nCount;
}

int  Erase(std::string& strTarget, char tar)
{
	size_t nPos = strTarget.npos; size_t nCount = 0;
	while ((nPos = strTarget.find(tar)) != strTarget.npos)
	{
		strTarget.erase(nPos,1);
		nCount++;
	}
	return nCount;
}

bool  IsMatch(const std::string& strSrc, const std::string& strTarget)
{
	return (strSrc.find(strTarget) != strSrc.npos) ? true : false;
}


  ```

# 19. QT_NO_CLIPBOARD

## 19.1 problem description

```
void test()
{
	int argc = 1; 
	char* argv[] = {"libnative-lib.so"};
	QApplication  application(argc,argv);
}


the function test() is broken when init the resources in Android Environment,

when debug that funtion 'QApplication  application(argc,argv);'

step into when hit the following function which has the Macro 'QT_NO_CLIPBOARD';

#ifndef QT_NO_CLIPBOARD
    m_androidPlatformClipboard = new QAndroidPlatformClipboard();
#endif

continue to step into 'new QAndroidPlatformClipboard()';
some jni methods has been called ,but we cannot find the method,so program  sink without any return ;
```
## 19.2 method


1. to find the method is the essential way;
2. but here we use another way in case that we have no so much information about the clipboard jni method;
   
   So if we do not call that logic function,just pass over it, the program do not sink;

### 19.2.1 HOW to pass over it?

we can define the Macro 'QT_NO_CLIPBOARD' to pass over it;

```
next we exam the method;

1. search the macro 'QT_NO_CLIPBOARD' in the source code folder;
   we get many hits in serveral modules ,such as gui | android platform | widget | modbus | assitant tools;
   but there is no hit about 'define QT_NO_CLIPBOARD';
   WHERE to deine the macro?

2. check some  *.h or *.cpp files in which hit the macro;
   find the common includes, '#include <QtGui/private/qtguiglobal_p.h>'
   
   in qtguiglobal_p.h ,then locate :
	#include <QtGui/qtguiglobal.h>
	#include <QtCore/private/qglobal_p.h>
	#include <QtGui/private/qtgui-config_p.h>
   
   in  qtguiglobal.h,locate:
	#include <QtCore/qglobal.h>
	#include <QtGui/qtgui-config.h>

	in qglobal.h or qglobal_p.h, we cannot find any macro defination;
	 
	for qtgui-config_p.h or qtgui-config.h, firstly we cannot find them in the source code folder;
	we find them in the compile folder which has been created when run the configure command;
	in qtgui-config_p.h or qtgui-config.h, we can find many macro definition;

3. we edit the file 'qtgui-config_p.h' by appending the definition of the macro  QT_NO_CLIPBOARD ;
   then compile the source code again and test the code;
   OK,we success; prove that that method is OK;

4. But  WHY we need edit the file?

```


### 19.2.2   WHY we need edit the file?
   is there another way to define the macro?

1. in file qtgui-config.h, when we search key words 'clipboard' ,
   we find '#define QT_FEATURE_clipboard 1';
   
   so what create the file qtgui-config.h and define FEATURE_clipboard?

2. in the folder which locate the file qtgui-config.h, we find file qtgui-config.pri;

```
	QT.gui.enabled_features = accessibility action opengles2 clipboard colornames cssparser cursor desktopservices imageformat_xpm draganddrop opengl imageformatplugin highdpiscaling im image_heuristic_mask image_text imageformat_bmp imageformat_jpeg imageformat_png imageformat_ppm imageformat_xbm movie pdf picture sessionmanager shortcut standarditemmodel systemtrayicon tabletevent texthtmlparser textodfwriter validator vulkan whatsthis wheelevent
	QT.gui.disabled_features = angle combined-angle-lib dynamicgl opengles3 opengles31 opengles32 openvg
	QT.gui.QT_CONFIG = accessibility action opengles2 clipboard colornames cssparser cursor desktopservices imageformat_xpm draganddrop opengl egl freetype imageformatplugin harfbuzz highdpiscaling ico im image_heuristic_mask image_text imageformat_bmp imageformat_jpeg imageformat_png imageformat_ppm imageformat_xbm movie pdf picture sessionmanager shortcut standarditemmodel systemtrayicon tabletevent texthtmlparser textodfwriter validator whatsthis wheelevent
```

	in file qtgui-config.pri,clipboard is the enabled feature;
	just list above;	

3. so what create the file qtgui-config.pri? How to make clipboard disabled ?

###	 19.2.3 How to make clipboard disabled ?

1. in source code folder, search key words 'clipboard' ,
2. locate file 'configure.json'; the configure script use this json file to configure;
3. in json file,
   CHANGE from 
   ```
	"clipboard": {
		"label": "QClipboard",
		"purpose": "Provides cut and paste operations.",
		"section": "Kernel",
		"condition": "!config.integrity && !config.qnx",
		"output": [ "publicFeature", "feature" ]
	},
	```
   into
	```
	"clipboard": {
		"label": "QClipboard",
		"purpose": "Provides cut and paste operations.",
		"section": "Kernel",
		"autoDetect": "config.msvc",
		"condition": "!config.integrity && !config.qnx",
		"output": [ "publicFeature", "feature" ]
	},
	```
4. save the changes and reconfigure,
   locate the file qtgui-config.h , there is no QT_FEATURE_clipboard defination;
   but we find '#define QT_NO_CLIPBOARD ';

5. bingo! we have made  FEATURE clipboard disabled;

6. recompile the source code and retest the demo;OK;
   


# 20. debug with add-dsym in AS

## 20.1 detail version

1. Compile the codes and Debug the app;
2. in Debug Tab,select app Tab,then select LLDB Tab;
   here we cannot attach the debug symbol;
3. in left dock buttons, click the pause button to pause the program,then select LLDB Tab;
4. in LLDB tab,attach the library symbols;just like:
   add-dsym E:/Qt/build/armeabi-v7a/qtbase/plugins/platforms/android/libqtforandroid.so
   if need to attach more than one library,call add-dsym one by one;
5. when we have add the target libraries, we can see the information:
    ```
   ‘symbol file 'E:\Qt\build\armeabi-v7a\qtbase\plugins\platforms\android\libqtforandroid.so' 
   has been added to 
   'C:\Users\Administrator\.lldb\module_cache\remote-android\.cache\A3E31994\libqtforandroid.so'’
    ```
6. in left dock buttons, click the resume button to resume the program;
7. then contine to debug the program; 
8. Step into the attached library;


## 20.2 short version

1. Debug ---> app --->LLDB
2. pause
3. add-dsym 
4. resume
5. debug

 ```
Executing commands in 'D:\android\Android Studio\bin\lldb\shared\stl_printers\load_script'.
(lldb) script import sys
(lldb) script import os
(lldb) script gala_available = os.environ.get('AS_GALA_PATH') is not None
(lldb) script exec("if gala_available: sys.path.append(os.environ['AS_GALA_PATH'])")
(lldb) script exec("if gala_available: import gdb")
(lldb) script exec("if gala_available: import gdb.printing")
(lldb) script libstdcxx_printers_available = gala_available and (os.environ.get('AS_LIBSTDCXX_PRINTER_PATH') is not None)
(lldb) script exec("if libstdcxx_printers_available: sys.path.append(os.environ['AS_LIBSTDCXX_PRINTER_PATH'])")
(lldb) script exec("if libstdcxx_printers_available: import printers")
(lldb) script exec("if libstdcxx_printers_available: printers.register_libstdcxx_printers(None)")
(lldb) script exec("if lldb.debugger.GetCategory('libstdc++-v6').IsValid(): lldb.debugger.GetCategory('gnu-libstdc++').SetEnabled(False)")
(lldb) add-dsym E:/Qt/build/armeabi-v7a/qtbase/plugins/platforms/android/libqtforandroid.so
symbol file 'E:\Qt\build\armeabi-v7a\qtbase\plugins\platforms\android\libqtforandroid.so' has been added to 'C:\Users\Administrator\.lldb\module_cache\remote-android\.cache\9BBA21BF\libqtforandroid.so'
(lldb) add-dsym E:/Qt/build/armeabi-v7a/qtbase/lib/libQt5Gui.so
symbol file 'E:\Qt\build\armeabi-v7a\qtbase\lib\libQt5Gui.so' has been added to 'C:\Users\Administrator\.lldb\module_cache\remote-android\.cache\8D29BD13\libQt5Gui.so'
 ```

# 21. init Qt resource

 ```
public class QtEnvironment {
    public static int Init(Activity activity) {
        String rootPath = android.os.Environment.getExternalStorageDirectory().getAbsolutePath();
        String apkPath = activity.getApplicationInfo().publicSourceDir;

        String nativeLibraryPrefix = activity.getApplicationInfo().nativeLibraryDir + "/";

        DexClassLoader classLoader = new DexClassLoader("", // .jar/.apk files
                activity.getDir("outdex", Context.MODE_PRIVATE).getAbsolutePath(), // directory where optimized DEX files should be written.
                null, // libs folder (if exists)
                activity.getClassLoader());

        QtNative.setClassLoader(classLoader);

        String[] qtLibs = {"Qt5Core","Qt5Gui","Qt5Widgets","qtforandroid"};
        ArrayList<String> libraryList = new ArrayList<String>();

        String libPrefix = nativeLibraryPrefix + "lib";
        for (int i = 0; i < qtLibs.length; i++)
            libraryList.add(libPrefix + qtLibs[i] + ".so");
        QtNative.loadQtLibraries(libraryList);

        return 1;
    }

}
 ```

# 22. Template specialization

 1. we use template function which has the original operation 
    to cover some or several cases;

    but when template function  crash some special case,
	the original operation cannot cover  ,

    we may specialize the template function with the speicial operation  ;

 2. in common ,function name is same in different cases,
    
	but with different parameter type and different operation;

 ```
template<typename T> bool i_FromString(T& t,std::string str)
{
	istringstream is(str);
	is.imbue(LOC_CHS);
	is >> t;
	return true;
}
template<size_t N> inline bool i_FromString(char (&sz)[N],std::string str)
{
	if (N>0)
	{
		memset(sz,0,sizeof(char)*N);
		strncpy(sz,str.c_str(),N-1);
		return true;
	}
	return false;
}
template<> inline bool i_FromString(char*& t,std::string str)
{
	const char* pChVal = str.c_str();
	lstrcpyn(t,pChVal,strlen(pChVal)+1);
	return true;
}

template<> inline bool i_FromString(float& t,std::string str)
{
	if(str != "")
	{
		float fVal = atof(str.c_str());
		memcpy(&t,&fVal,sizeof(float));
		return true;
	}
	return false;
}

template<> inline bool i_FromString(double& t,std::string str)
{
	if(str != "")
	{
#if defined(BUTTER_ANDROID)
		double dVal = atof(str.c_str());
		memcpy(&t,&dVal,sizeof(double));
#else
		t = atof(str.c_str());
#endif
		return true;
	}
	return false;
}

template<> inline bool i_FromString(GUID& t,std::string str)
{
	CString   strTemp;
	if(str != "")
	{
		strTemp = str.c_str();
		CvtStringToGuid(strTemp,t);
		return true;
	}
	return false;
}

template<> inline bool i_FromString(string& t,std::string str)
{
	t = str;
	return true;
}
template<> inline bool i_FromString(CString& t, std::string str)
{
	t = str.c_str();
	return true;
}
 ```


# 23. Howt to access unaligned memory

  * Unaligned memory accesses occur when you try to read N bytes of data starting from an address 
    
	that is not evenly divisible by N (i.e. addr % N != 0).
  
    For example, reading 4 bytes of data from address 0x10004 is fine, 
	
	but reading 4 bytes of data from address 0x10005 would be an unaligned memory access.
   
* when access the unligned memory directly,OS may sent SIGBUS error;
   
* if read the unligned memory, use function 'get_unaligned(ptr)'
  
  if write the unligned  memory,use memcpy();

* for more information ,please visit https://www.kernel.org/doc/html/latest/core-api/unaligned-memory-access.html


##  case 1: both dt and st is aligned
 ```
## for case: both dt and st is aligned
template<typename T> void i_AssignValue(T& dt, T& st)
{
	i_FromString(dt, i_ToString(st));
}
 ```
##  case 2: either dt or st is aligned or both are unligned


 ```
/************************************************************
 *  for case: either dt or st is unaligned;
 *  if read the unligned value, use function 'get_unaligned(ptr)'
 *  if write the unligned value,use memcpy();
*************************************************************/

#ifdef BUTTER_ANDROID
#ifndef __GET_UNALIGNED_
#define __GET_UNALIGNED_

#define __get_unaligned_t(type, ptr) ({						\
	const struct { type x; } __packed *__pptr = (__typeof(__pptr))(ptr);	\
	__pptr->x;								\
})

#define get_unaligned(ptr)	__get_unaligned_t(__typeof(*(ptr)), (ptr))

#endif /* __GET_UNALIGNED_ */

/// for case: Ether dt or st is unaligned
template<> inline void i_AssignValue(float& dt, float& st)
{
	float val;
	const void* ptr = (void*)&st;
	//uint16_t temp16 = get_unaligned((uint16_t *) ptr); //OK
	float temp16 = get_unaligned((float *) ptr); /// read the unligned memory
	std::string strVal = i_ToString((double)temp16);
	i_FromString<float>(val,strVal);
	memcpy(&dt, &val, sizeof(float)); /// write the unligned memory
}

#endif //BUTTER_ANDROID

```

# 24. Unable to open DISPLAY
```
java.lang.reflect.InvocationTargetException
Casued by:
java.lang.unsupportedOperationException:Unable to open DISPLAY

```
## solution
* echo $DISPLAY
* export DISPLAY=:1
* xhost +
* run sh run.sh
  
# 25. compute time cost
```

int main()
{
	{
		CHTime timecost;
		fun1();
	}
	
	{
		CHTime timecost;
		fun2();
		timecost.getTime();
	}
}

////**********************CHTime.h************************

#ifdef WIN32
/// for windows
#include <windows.h>　
#else //BUTTER_LINUX
//// for linux
#include <sys/time.h>　
#endif

class CHTime
{
public:
	CHTime(){ 
		Begin();
	}
	~CHTime(){
		End();
	}
	void Begin()
	{
#ifdef WIN32
       QueryPerformanceFrequency(&m_freq);
       QueryPerformanceCounter(&m_queryBegin);
#else
       gettimeofday(&m_begin,NULL);
#endif
	}
	void End()
	{
#ifdef WIN32   
       QueryPerformanceCounter(&m_queryEnd);
       m_timeuse=(double)(m_queryEnd.QuadPart-m_queryBegin.QuadPart)/(double)m_freq.QuadPart * 1000.0; 
#else
       gettimeofday(&m_end,NULL);
       m_timeuse = (m_end.tv_sec - m_begin.tv_sec) + (double)(m_end.tv_usec - m_begin.tv_usec)/1000.0;
#endif
      cout<<"timecost = "<<m_timeuse<<endl;  //（unit：ms）
	}
	double getTime()
	{
		End(); 
		return m_timeuse;
	}
protected:	
    double m_timeuse;///ms

#ifdef WIN32
    LARGE_INTEGER m_queryBegin,m_queryEnd,m_freq;
#else
	struct timeval m_begin,m_end;
#endif

}


```
# 26. how to get tianditu token
![get tianditu token](./img/26_tiandituToken.png)

STEPS:
 * 1. visit 'tianditu.gov.cn' in chrome browser;
 * 2. Press 'F12'and show the right window;
 * 3. click and drag or move or zoom in or zoom out the left map;
      some information will list in the right window;
 * 4. choose the 'NetWork' tab,in 'Name' window list the requests,choose any one;
 * 5. in Headers tab, 'General' Node,we get the URL:
  
      https://t6.tianditu.gov.cn/vec_w/wmts?SERVICE=WMTS&REQUEST=GetTile&VERSION=1.0.0&LAYER=vec&STYLE=default&TILEMATRIXSET=w&FORMAT=tiles&TILECOL=55&TILEROW=24&TILEMATRIX=6&tk=9a02b3cdd29cd346de4df04229797710
* 6. in the above URL,get the value of the key 'tk' 
    
	 Ok, we get the tianditu token: 9a02b3cdd29cd346de4df04229797710

# 27. compile cario

## For linux, Steps:

1. get source codes 
	```
	from http://www.cairographics.org/releases/pixman-0.40.0.tar.gz  
	get pixman library;

	from https://cairographics.org/snapshots/cairo-1.17.2.tar.xz
	get cario library;

	or visit url: https://www.cairographics.org/releases/ 
	select the latest version;
	```
2. then copy the source codes into some folder,
   like: /home/src/code/ ;

   in shell terminal,use commands below to decompress the tar file; 

		tar zxvf pixman-0.40.0.tar.gz 
		tar zxvf cairo-1.17.2.tar.xz 

3. compile pixman
  
      open the pixman source code root folder,then configure、 make、install
		
		cd /home/src/code/pixman-0.40.0

		./configure --prefix=/home/src/code/pixman 

		make
		
		make install
		
4. compile cario
      
	4.1 open the cario source code root folder, then configure、 make、install

		cd /home/src/code/cairo-1.17.2

		./configure --prefix=/home/src/code/cario --enable-ps pixman_CFLAGS='-I/home/src/code/pixman/include/pixman-1' pixman_LIBS='-L/home/src/code/pixman/lib -lpixman-1'  png_CFLAGS='-I/home/os/git/ins/png/include' png_LIBS='-L/home/os/git/ins/png/lib  -lpng16'

		make

		make install
	4.2 how to get the dependent librarys information
	    
		use ' ./configure --help ' get some environment variables，like below;

	```	
	png_CFLAGS  C compiler flags for png, overriding pkg-config
	png_LIBS    linker flags for png, overriding pkg-config 
	gl_CFLAGS   C compiler flags for gl, overriding pkg-config
	gl_LIBS     linker flags for gl, overriding pkg-config
	FREETYPE_CFLAGS
			C compiler flags for FREETYPE, overriding pkg-config
	FREETYPE_LIBS
			linker flags for FREETYPE, overriding pkg-config
	FONTCONFIG_CFLAGS
			C compiler flags for FONTCONFIG, overriding pkg-config
	FONTCONFIG_LIBS
			linker flags for FONTCONFIG, overriding pkg-config
	LIBRSVG_CFLAGS
			C compiler flags for LIBRSVG, overriding pkg-config
	LIBRSVG_LIBS
			linker flags for LIBRSVG, overriding pkg-config
	pixman_CFLAGS
			C compiler flags for pixman, overriding pkg-config
	pixman_LIBS linker flags for pixman, overriding pkg-config
	```
	
	set the environment variables:
	```
	pixman_CFLAGS='-I/home/src/code/pixman/include/pixman-1' 
	pixman_LIBS='-L/home/src/code/pixman/lib -lpixman-1'
	```
5. get cario library
   
	```
	open folder /home/src/code/cario ,get libs
	```
## For Android, Steps:

	export CPPFLAGS=--sysroot="/opt/android-ndk-r13b/platforms/android-23/arch-arm/ "
	export CFLAGS=--sysroot="/opt/android-ndk-r13b/platforms/android-23/arch-arm/ "
	export CXXFLAGS=--sysroot="/opt/android-ndk-r13b/platforms/android-23/arch-arm/ "
	export LDFLAGS=--sysroot="/opt/android-ndk-r13b/platforms/android-23/arch-arm/ "
	export CC="/opt/android-ndk-r13b/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64/bin/arm-linux-androideabi-gcc "
	export CXX="/opt/android-ndk-r13b/toolchains/arm-linux-androideabi-4.9/prebuilt/linux-x86_64/bin/arm-linux-androideabi-g++ "

	./configure --host=arm --prefix=/opt/cairo/cairo --with-sysroot="/opt/android-ndk-r13b/platforms/android-23/arch-arm/" --enable-png=yes --disable-xlib --disable-xlib-xrender --enable-directfb=no --enable-ft=yes  --enable-fc=no --disable-win32 --disable-svg --enable-pdf=yes  FREETYPE_CFLAGS='-I/opt/cairo/freetype/include/freetype2' FREETYPE_LIBS='-L/opt/cairo/freetype/lib -lfreetype' png_CFLAGS='-I/opt/cairo/png/include' png_LIBS='-L/opt/cairo/png/lib -lpng16' pixman_CFLAGS='-I/opt/cairo/pix_man/include/pixman-1' pixman_LIBS='-L/opt/cairo/pix_man/lib -lpixman-1' --enable-xcb=no 

## For IOS, Steps:

	#export PKG_CONFIG_PATH=/usr/lib/pkgconfig/
	export BUILD_ARCHS=armv7 armv7s arm64 i386 x86_64
	#IOS_LINK_FLAG= -arch x86_64
	export IOS_BUILD_TOOLS_DIR=/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/
	export IOS_SYSROOT_DIR="/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk"
	#export CC="/Applications/Xcode.app/Contents/Developer/usr/bin/gcc "
	#export CXX="/Applications/Xcode.app/Contents/Developer/usr/bin/g++ "

	#export CC="/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang "
	#export CXX="/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/clang++ "

	./configure --prefix=/opt/simulator_cairo_lib/cairo CC="/Applications/Xcode.app/Contents/Developer/usr/bin/gcc -arch x86_64" --with-sysroot="/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk" AR="/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/ar" --enable-xlib=no --enable-xlib-xrender=no --enable-ft=no --enable-script=no --enable-ps=no --enable-pdf=no --enable-svg=no --enable-trace=no --enable-interpreter=no --enable-png=no  pixman_CFLAGS='-I/opt/simulator_cairo_lib/pixmap/include/pixman-1' pixman_LIBS='-L/opt/simulator_cairo_lib/pixmap/lib -lpixman-1' png_CFLAGS='-I/opt/simulator_cairo_lib/png/include' png_LIBS='-L/opt/simulator_cairo_lib/png/lib/ -lpng16' 

# 28.QGuiApplication(int &argc, char **argv)

 for more information, please visit 

   https://doc.qt.io/qt-5/qguiapplication.html#QGuiApplication

   https://stackoverflow.com/questions/17979185/qt-5-1-qapplication-without-display-qxcbconnection-could-not-connect-to-displ

Initializes the window system and constructs an application object with argc command line arguments in argv.

Warning: The data referred to by argc and argv must stay valid for the entire lifetime of the QGuiApplication object. In addition, argc must be greater than zero and argv must contain at least one valid character string.

The global qApp pointer refers to this application object. Only one application object should be created.

This application object must be constructed before any paint devices (including pixmaps, bitmaps etc.).

Note: argc and argv might be changed as Qt removes command line arguments that it recognizes.

## Supported Command Line Options

All Qt programs automatically support a set of command-line options that allow modifying the way Qt will interact with the windowing system. Some of the options are also accessible via environment variables, which are the preferred form if the application can launch GUI sub-processes or other applications (environment variables will be inherited by child processes). When in doubt, use the environment variables.

### The options currently supported are the following:

```
-platform platformName[:options], specifies the Qt Platform Abstraction (QPA) plugin.
Overrides the QT_QPA_PLATFORM environment variable.
```

```
-platformpluginpath path, specifies the path to platform plugins.
Overrides the QT_QPA_PLATFORM_PLUGIN_PATH environment variable.

-platformtheme platformTheme, specifies the platform theme.
Overrides the QT_QPA_PLATFORMTHEME environment variable.

-plugin plugin, specifies additional plugins to load. The argument may appear multiple times.
Concatenated with the plugins in the QT_QPA_GENERIC_PLUGINS environment variable.

-qmljsdebugger=, activates the QML/JS debugger with a specified port. The value must be of format port:1234[,block], where block is optional and will make the application wait until a debugger connects to it.
-qwindowgeometry geometry, specifies window geometry for the main window using the X11-syntax. For example: -qwindowgeometry 100x100+50+50
-qwindowicon, sets the default window icon
-qwindowtitle, sets the title of the first window
-reverse, sets the application's layout direction to Qt::RightToLeft. This option is intended to aid debugging and should not be used in production. The default value is automatically detected from the user's locale (see also QLocale::textDirection()).
-session session, restores the application from an earlier session.
The following standard command line options are available for X11:

-display hostname:screen_number, switches displays on X11.
Overrides the DISPLAY environment variable.

-geometry geometry, same as -qwindowgeometry.
Platform-Specific Arguments
You can specify platform-specific arguments for the -platform option. Place them after the platform plugin name following a colon as a comma-separated list. For example, -platform windows:dialogs=xp,fontengine=freetype.
```


### platformName : const QString
This property holds the name of the underlying platform plugin.

The QPA platform plugins are located in qtbase\src\plugins\platforms. At the time of writing, the following platform plugin names are supported:
```
android
cocoa is a platform plugin for macOS.
directfb
eglfs is a platform plugin for running Qt5 applications on top of EGL and OpenGL ES 2.0 without an actual windowing system (like X11 or Wayland). For more information, see EGLFS.
ios (also used for tvOS)
kms is an experimental platform plugin using kernel modesetting and DRM (Direct Rendering Manager).
linuxfb writes directly to the framebuffer. For more information, see LinuxFB.
minimal is provided as an examples for developers who want to write their own platform plugins. However, you can use the plugin to run GUI applications in environments without a GUI, such as servers.
minimalegl is an example plugin.
offscreen
openwfd
qnx
windows
wayland is a platform plugin for modern Linux desktops and some embedded systems.
xcb is the X11 plugin used on regular desktop Linux platforms.
For more information about the platform plugins for embedded Linux devices, see Qt for Embedded Linux.

Access functions:

QString	platformName()
```

### The following parameters are available for -platform windows:

```
altgr, detect the key AltGr found on some keyboards as Qt::GroupSwitchModifier (since Qt 5.12).
darkmode=[1|2] controls how Qt responds to the activation of the Dark Mode for applications introduced in Windows 10 1903 (since Qt 5.15).
A value of 1 causes Qt to switch the window borders to black when Dark Mode for applications is activated and no High Contrast Theme is in use. This is intended for applications that implement their own theming.

A value of 2 will in addition cause the Windows Vista style to be deactivated and switch to the Windows style using a simplified palette in dark mode. This is currently experimental pending the introduction of new style that properly adapts to dark mode.

dialogs=[xp|none], xp uses XP-style native dialogs and none disables them.
dpiawareness=[0|1|2] Sets the DPI awareness of the process (see High DPI Displays, since Qt 5.4).
fontengine=freetype, uses the FreeType font engine.
menus=[native|none], controls the use of native menus.
Native menus are implemented using Win32 API and are simpler than QMenu-based menus in for example that they do allow for placing widgets on them or changing properties like fonts and do not provide hover signals. They are mainly intended for Qt Quick. By default, they will be used if the application is not an instance of QApplication or for Qt Quick Controls 2 applications (since Qt 5.10).

nocolorfonts Turn off DirectWrite Color fonts (since Qt 5.8).
nodirectwrite Turn off DirectWrite fonts (since Qt 5.8).
nomousefromtouch Ignores mouse events synthesized from touch events by the operating system.
nowmpointer Switches from Pointer Input Messages handling to legacy mouse handling (since Qt 5.12).
reverse Activates Right-to-left mode (experimental). Windows title bars will be shown accordingly in Right-to-left locales (since Qt 5.13).
tabletabsoluterange=<value> Sets a value for mouse mode detection of WinTab tablets (Legacy, since Qt 5.3).
The following parameter is available for -platform cocoa (on macOS):

fontengine=freetype, uses the FreeType font engine.
For more information about the platform-specific arguments available for embedded Linux platforms, see Qt for Embedded Linux.
```

# 29.lib-config.cmake

*  set(xxx_FOUND TRUE)
*  use xxx_INCLUDE_DIRS and xxx_LIBRARIES in CMakeLists.txt
*  set(xxx_INCLUDE_DIRS ${MYSQLCPPCONN_INCLUDE_DIR} )
*  set(xxx_LIBRARIES ${MYSQLCPPCONN_LIBRARY} )

 ##  pslib-config.cmake
```
get_filename_component(_DIR "${CMAKE_CURRENT_LIST_FILE}" PATH)

if(NOT PSLIB_FIND_COMPONENTS)
    set(PSLIB_FIND_COMPONENTS pslib pslib)
    if(PSLIB_FIND_REQUIRED)
        set(PSLIB_FIND_REQUIRED_pslib TRUE)
    endif()

    set(PSLIB_FOUND TRUE)
endif()

set(PSLIB_INCLUDE_DIRS ${_DIR}/../../include)
set(PSLIB_LIBRARIES)
if (EXISTS ${_DIR}/../../lib/pslib.a)
    list(APPEND PSLIB_LIBRARIES optimized ${_DIR}/../../lib/pslib.a)
endif()
if (EXISTS ${_DIR}/../../debug/lib/pslib.a)
    list(APPEND PSLIB_LIBRARIES debug ${_DIR}/../../debug/lib/pslib.a)
endif()
if (EXISTS ${_DIR}/../../lib/pslib.lib)
    list(APPEND PSLIB_LIBRARIES optimized ${_DIR}/../../lib/pslib.lib)
endif()
if (EXISTS ${_DIR}/../../debug/lib/pslib.lib)
    list(APPEND PSLIB_LIBRARIES debug ${_DIR}/../../debug/lib/pslib.lib)
endif()
```
 ##  mysqlcppconn-config.cmake
```
# Name: <Name>Config.cmake  or <lower name>-config.cmake
# mysqlcppconn-config.cmake or MYSQLCPPCONNConfig.cmake  
# similar to OpenCVConfig.cmake   

# Tips for MYSQLCPPCONN_ROOT_DIR
# use "C:/Program Files/MySQL/Connector.C++ 1.1", otherwise cmake-gui can not auto find include and library

set(MYSQLCPPCONN_FOUND TRUE) # auto 
set(MYSQLCPPCONN_ROOT_DIR "C:/Program Files/MySQL/Connector.C++ 1.1")

find_path(MYSQLCPPCONN_INCLUDE_DIR NAMES cppconn/driver.h PATHS "${MYSQLCPPCONN_ROOT_DIR}/include") 
mark_as_advanced(MYSQLCPPCONN_INCLUDE_DIR) # show entry in cmake-gui

find_library(MYSQLCPPCONN_LIBRARY NAMES mysqlcppconn.lib PATHS "${MYSQLCPPCONN_ROOT_DIR}/lib/opt") 
mark_as_advanced(MYSQLCPPCONN_LIBRARY) # show entry in cmake-gui

# use xxx_INCLUDE_DIRS and xxx_LIBRARIES in CMakeLists.txt
set(MYSQLCPPCONN_INCLUDE_DIRS ${MYSQLCPPCONN_INCLUDE_DIR} )
set(MYSQLCPPCONN_LIBRARIES ${MYSQLCPPCONN_LIBRARY} )

# cmake entry will be saved to build/CMakeCache.txt 

message( "mysqlcppconn-config.cmake " ${MYSQLCPPCONN_ROOT_DIR})
```
## halcon-config.cmake
```
# halcon-config.cmake or HALCONConfig.cmake  

set(HALCON_FOUND TRUE) # auto 
set(HALCON_ROOT_DIR E:/git/car/windows/lib/halcon)

find_path(HALCON_INCLUDE_DIR NAMES halconcpp/HalconCpp.h PATHS "${HALCON_ROOT_DIR}/include") 
mark_as_advanced(HALCON_INCLUDE_DIR) # show entry in cmake-gui

find_library(HALCON_LIBRARY NAMES halconcpp.lib PATHS "${HALCON_ROOT_DIR}/lib/x64-win64") 
mark_as_advanced(HALCON_LIBRARY) # show entry in cmake-gui

# use xxx_INCLUDE_DIRS and xxx_LIBRARIES in CMakeLists.txt
set(HALCON_INCLUDE_DIRS ${HALCON_INCLUDE_DIR} )
set(HALCON_LIBRARIES ${HALCON_LIBRARY} )

message( "halcon-config.cmake " ${HALCON_ROOT_DIR})
```
## usage
```
find_package(HALCON REQUIRED) # user-defined
include_directories(${HALCON_INCLUDE_DIRS})

find_package(MYSQLCPPCONN REQUIRED) # user-defined
include_directories(${MYSQLCPPCONN_INCLUDE_DIRS})

target_link_libraries(demo ${HALCON_LIBRARIES} ${MYSQLCPPCONN_LIBRARIES})
```

## cmake-gui entry
1.  start cmake-gui, and at first,we should set

* HALCON_DIR = E:/git/car/share/cmake-3.10/Modules
* MYSQLCPPCONN_DIR = E:/git/car/share/cmake-3.10/Modules
2.  then configure

  * HALCON_INCLUDE_DIR and HALCON_LIBRARY will be found.
  * MYSQLCPPCONN_INCLUDE_DIR and MYSQLCPPCONN_LIBRARY will be found.
  
# 30.tianditu-GetCapabilities

   http://t0.tianditu.gov.cn/vec_c/wmts?tk=9a02b3cdd29cd346de4df04229797710&request=GetCapabilities&service=wmts


# 31.Install Zentao for linux

##  Install steps

* 1. visit https://www.zentao.net/dynamic/zentaopms.pro10.3.1-80434.html ,

      get the linux install package,eg. 'ZenTaoPMS.15.7.1.zbox_64.tar.gz '
* 2. copy the package file into linux file folder,such as ' /user/zen '
  
* 3. open folder where package located, 'cd /user/zen '
  
* 4. unzip the file to target folder  '/opt/'; 
    
			 sudo tar -zxvf  ZenTaoPMS.15.7.1.zbox_64.tar.gz -C /opt

	  then in folder ' /opt/zbox ',we find ./zbox entry;

* 5. set port number for mysql and apache;
     
			cd /opt/zbox 
			./zbox  -mp 3307 
			./zbox  -ap 9091 
	 mp for mysql port;  ap for apache port
	 
* 6. add user into DataBase
     
	     cd /opt/zbox/auth
	     sh adduser.sh 

		 This tool is used to add user to access adminer
		 Account: root
		 Password: Adding password for user root

* 7. start mysql and apache
          
		 cd /opt/zbox
		 ./zbox start

* 8. visit 'localhost:9091 ' in local machine or
    
	 visit '192.168.10.50:9091 ' in remote machine;

	 then login and use it;

	 default user :   admin 

	 default key :    123456

* 9. when no longer use it,stop it;
     when reboot,restart it;

		 cd /opt/zbox
		 ./zbox stop
		 ./zbox restart	 
		 
* 10. if cannot visit，add port into firewall,if the firewall is active
      
			[root@localhost zbox]# systemctl status firewalld
			● firewalld.service - firewalld - dynamic firewall daemon
			   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
			   Active: active (running) since 日 2021-12-05 11:36:48 CST; 1h 47min ago
				 Docs: man:firewalld(1)
			 Main PID: 846 (firewalld)
				Tasks: 2
			   CGroup: /system.slice/firewalld.service
					   └─846 /usr/bin/python2 -Es /usr/sbin/firewalld --nofork --nopid

			12月 05 11:36:46 localhost.localdomain systemd[1]: Starting firewalld - dynam...
			12月 05 11:36:48 localhost.localdomain systemd[1]: Started firewalld - dynami...
			Hint: Some lines were ellipsized, use -l to show in full.
			[root@localhost zbox]# firewall-cmd --list-ports
			80/tcp 8081/tcp
			[root@localhost zbox]# firewall-cmd --zone=public --add-port=9091/tcp --permanentsuccess
			[root@localhost zbox]# firewall-cmd --list-ports
			80/tcp 8081/tcp
			[root@localhost zbox]# reboot  

# 32.Install Gitlab for linux

## About gitlab

	/etc/gitlab/gitlab.rb          #gitlab配置文件
	/opt/gitlab                    #gitlab的程序安装目录
	/var/opt/gitlab                #gitlab目录数据目录
	/var/opt/gitlab/git-data       #存放仓库数据
	gitlab-ctl reconfigure         #重新加载配置
	gitlab-ctl status              #查看当前gitlab所有服务运行状态
	gitlab-ctl stop                #停止gitlab服务
	gitlab-ctl stop nginx          #单独停止某个服务
	gitlab-ctl tail                #查看所有服务的日志

	Gitlab的服务构成：
	nginx：                        静态web服务器
	gitlab-workhorse               轻量级反向代理服务器
	logrotate                      日志文件管理工具
	postgresql                     数据库
	redis                          缓存数据库
	sidekiq                        用于在后台执行队列任务（异步执行）

##  Install steps

* 1. prepare 
   * centos 7 , 4G memory
   * gitlab-ce-10.2.2-ce
   * xftp ,xshell
  
* 2. visit https://mirrors.tuna.tsinghua.edu.cn/gitlab-ee/yum/el7/ ,
     
	 get the install package gitlab-ee-14.5.1-ee.0.el7.x86_64.rpm
    
	 then copy package to the target folder /user/gitlab in centos7 ;

* 3. install dependency and package 
   
         yum install -y postfix sshd policycoreutils-python

		 cd /user/gitlab

	     rpm -ivh gitlab-ee-14.5.1-ee.0.el7.x86_64.rpm

* 4. configure
    
			[root@gitlab ~]# vim /etc/gitlab/gitlab.rb     
			external_url 'http://192.168.1.21'        
			[root@gitlab ~]# gitlab-ctl reconfigure 

* 5. visit gitlab 
  
			in remote machine visit the ip configured in step 4;
			http://192.168.1.21

* 6. change ui chinese

		1、dowload the service pack 
		[root@gitlab ~]# git clone https://gitlab.com/xhang/gitlab.git
		[root@gitlab ~]# cd gitlab    

		2、find the branch version
		[root@gitlab ~]# git branch -a

		3、compare version ,generate the service pack
		[root@gitlab ~]# git diff remotes/origin/10-2-stable remotes/origin/10-2-stable-zh > /tmp/10.2.2-zh.diff

		4、stop gitlab service
		[root@gitlab ~]# gitlab-ctl stop

		5、patch the service pack
		[root@gitlab ~]# patch -d /opt/gitlab/embedded/service/gitlab-rails -p1 < /tmp/10.2.2-zh.diff

		6、start and reconfigure
		[root@gitlab ~]# gitlab-ctl start
		[root@gitlab ~]# gitlab-ctl reconfigure

* 7.  fire wall
   
      if cannot visit gitlab,please close fire wall or open port 80;
    
			service iptables stop

			vi /etc/sysconfig/iptables

			-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT

			service iptables restart

* 8. config email

			vim /etc/gitlab/gitlab.rb  
			external_url 'http://192.168.1.21' 
			### Email Settings
			gitlab_rails['gitlab_email_enabled'] = true
			gitlab_rails['gitlab_email_from'] = 'huawei@qq.com'
			gitlab_rails['gitlab_email_display_name'] = 'huawei'

			### GitLab email server settings
			### here use the qq email,if use other email server,modify the smtp address
			gitlab_rails['smtp_enable'] = true
			gitlab_rails['smtp_address'] = "smtp.exmail.qq.com"
			gitlab_rails['smtp_port'] = 465
			gitlab_rails['smtp_user_name'] = "huawei@qq.com"
			gitlab_rails['smtp_password'] = "123456"
			gitlab_rails['smtp_authentication'] = "login"
			gitlab_rails['smtp_enable_starttls_auto'] = true
			gitlab_rails['smtp_tls'] = true
			
* 9. login with root

     username: root
	 password: QjD1J3CZW3XOnJnWX5ieUt7V6GoSJjskvQiBNPnxBDc=
	 password get from file 'initial_root_password' which located in ' /etc/gitlab'

		[root@localhost gitlab]# pwd
		/etc/gitlab
		[root@localhost gitlab]# ls
		gitlab.rb  gitlab-secrets.json  initial_root_password  trusted-certs
		[root@localhost gitlab]# cat initial_root_password 
		# WARNING: This value is valid only in the following conditions
		#          1. If provided manually (either via `GITLAB_ROOT_PASSWORD` environment variable or via `gitlab_rails['initial_root_password']` setting in `gitlab.rb`, it was provided before database was seeded for the first time (usually, the first reconfigure run).
		#          2. Password hasn't been changed manually, either via UI or via command line.
		#
		#          If the password shown here doesn't work, you must reset the admin password following https://docs.gitlab.com/ee/security/reset_user_password.html#reset-your-root-password.

		Password: QjD1J3CZW3XOnJnWX5ieUt7V6GoSJjskvQiBNPnxBDc=

		# NOTE: This file will be automatically deleted in the first reconfigure run after 24 hours.
		[root@localhost gitlab]#


# 33. defines.h

	/***********************************************************************
	* Software License Agreement (BSD License)
	*
	* Copyright 2020-2021  cheldon (program68@126.com). All rights reserved.
	*
	* Redistribution and use in source and binary forms, with or without
	* modification, are permitted provided that the following conditions
	* are met:
	*
	* 1. Redistributions of source code must retain the above copyright
	*    notice, this list of conditions and the following disclaimer.
	* 2. Redistributions in binary form must reproduce the above copyright
	*    notice, this list of conditions and the following disclaimer in the
	*    documentation and/or other materials provided with the distribution.
	*
	* THIS SOFTWARE IS PROVIDED BY THE AUTHOR ``AS IS'' AND ANY EXPRESS OR
	* IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES
	* OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED.
	* IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY DIRECT, INDIRECT,
	* INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT
	* NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
	* DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
	* THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
	* (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF
	* THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
	*************************************************************************/

	#ifndef BUTTER_DEFINES_H_
	#define BUTTER_DEFINES_H_

	
	#ifdef BUTTER_VERSION_
	#undef BUTTER_VERSION_
	#endif
	#define BUTTER_VERSION_ "1.2.5"

	#ifdef BUTTER_EXPORT
	#undef BUTTER_EXPORT
	#endif
	#ifdef WIN32
	/* win32 dll export/import directives */
	#ifdef BUTTER_EXPORTS
	#define BUTTER_EXPORT __declspec(dllexport)
	#elif defined(BUTTER_STATIC)
	#define BUTTER_EXPORT
	#else
	#define BUTTER_EXPORT __declspec(dllimport)
	#endif
	#else
	/* unix needs nothing */
	#define BUTTER_EXPORT
	#endif

	#ifdef BUTTER_DEPRECATED
	#undef BUTTER_DEPRECATED
	#endif
	#ifdef __GNUC__
	#define BUTTER_DEPRECATED __attribute__ ((deprecated))
	#elif defined(_MSC_VER)
	#define BUTTER_DEPRECATED __declspec(deprecated)
	#else
	#pragma message("WARNING: You need to implement BUTTER_DEPRECATED for this compiler")
	#define BUTTER_DEPRECATED
	#endif

	#undef BUTTER_PLATFORM_64_BIT
	#undef BUTTER_PLATFORM_32_BIT
	#if __amd64__ || __x86_64__ || _WIN64 || _M_X64
	#define BUTTER_PLATFORM_64_BIT
	#else
	#define BUTTER_PLATFORM_32_BIT
	#endif

	#undef BUTTER_ARRAY_LEN
	#define BUTTER_ARRAY_LEN(a) (sizeof(a)/sizeof(a[0]))

	#ifdef __cplusplus
	namespace butter {
	#endif

	enum butter_log_level
	{
		BUTTER_LOG_NONE = 0,
		BUTTER_LOG_FATAL = 1,
		BUTTER_LOG_ERROR = 2,
		BUTTER_LOG_WARN = 3,
		BUTTER_LOG_INFO = 4,
		BUTTER_LOG_DEBUG = 5
	};


	#ifdef __cplusplus
	}
	#endif


	#endif /* BUTTER_DEFINES_H_ */


# 34.error: invalid token at start of a preprocessor expression

for more information,visit

 https://stackoverflow.com/questions/36317145/macro-in-objective-c-calling-isequaltostring-produces-error-about-invalid-token


	error: invalid token at start of a preprocessor expression
	#if __amd64__ || __x86_64__ || _WIN64 || _M_X64
	#endif   
        

in some place; define _WIN64

    #define _WIN64

Remove this line ;

or change to

	#ifdef __amd64__ || __x86_64__ || _WIN64 || _M_X64
	#endif 



# 35. Predefined MACROS from compiler、HardWare、OS

## 《Calling conventions for different C++ compilers and operating systems》

for more information ,

please visit https://www.agner.org/optimize/calling_conventions.pdf  [page 56]

or

search key words "《Calling conventions for different C++ compilers and operating systems》"

Most C++ compilers have a predefined macro containing the version number of the
compiler. Programmers can use preprocessing directives to check for the existence of these
macros in order to detect which compiler the program is compiled on and thereby fix
problems with incompatible compilers.



## Table 1. Compiler version predefined macros


	Compiler             |        Predefined macro
	Borland              |        __BORLANDC__
	Codeplay VectorC     |        __VECTORC__
	Digital Mars         |        __DMC__
	Gnu                  |        __GNUC__
	Intel                |        __INTEL_COMPILER
	Microsoft            |        _MSC_VER
	Pathscale            |        __PATHSCALE__
	Symantec             |        __SYMANTECC__
	Watcom               |        __WATCOMC__


Unfortunately, not all compilers have well - documented macros telling which hardware
platform and operating system they are compiling for.The following macros may or may not
be defined :

## Table 2. Hardware platform predefined macros

	platform              |       Predefined macro
	x86                   |      _M_IX86  __INTEL__  __i386__
	x86-64                |      _M_X64   __x86_64__  __amd64
	IA64                  |      __IA64__
	DEC Alpha             |      __ALPHA__
	Motorola Power PC     |      __POWERPC__
	Any little endian     |      __LITTLE_ENDIAN__
	Any big endian        |      __BIG_ENDIAN__


## Table 3. Operating system predefined macros

	Operating system       |    Predefined macro
	DOS 16 bit             |    __MSDOS__   _MSDOS
	Windows 16 bit         |    _WIN16
	Windows 32 bit         |    _WIN32      __WINDOWS__
	Windows 64 bit         |    _WIN64      _WIN32
	Linux 32 bit           |    __unix__    __linux__
	Linux 64 bit           |    __unix__    __linux__ __LP64__ __amd64
	BSD                    |    __unix__    __BSD__   __FREEBSD__
	Mac OS                 |    __APPLE__   __DARWIN__ __MACH__
	OS / 2                 |    __OS2__


# 36. process character

	void process(const char* lpszString)
	{
		const char*		lpszBegin	= lpszString;
		const char*		lpszEnd		= NULL;

		// skip space 
		while(::isspace(*lpszBegin))
			lpszBegin = ::_tcsinc(lpszBegin);

		// begine with '<' ?
		if(*lpszBegin != _T('<'))
			return 0;

		// skip'<'
		lpszBegin = ::_tcsinc(lpszBegin);

		// if empty ,return
		if(*lpszBegin == _T('>'))
		   return;

         bool bClosingTag =false;
		// check label is character or not
		if(!::isalpha(*lpszBegin))
		{
			bClosingTag = (*lpszBegin==_T('/'));
			if(bClosingTag)
				lpszBegin = ::_tcsinc(lpszBegin);
			else
				return 0;
		}

		bOpeningTag = !bClosingTag;
		lpszEnd = lpszBegin;
		do
		{
			// include (a-z, A-Z) \(0-9) \'-' \'-' \':' \'.'
			if((!::isalnum(*lpszEnd)) && 
				(*lpszEnd!=_T('-')) && (*lpszEnd!=_T(':')) && 
				(*lpszEnd!=_T('_')) && (*lpszEnd!=_T('.')))
			{
				// if illegal
				ASSERT(lpszEnd != lpszBegin);

				// check state
				if(*lpszEnd==NULL || ::isspace(*lpszEnd) || 
					*lpszEnd == _T('>') || (*lpszEnd == _T('/') && 
					(!bClosingTag)))
				{
					break;
				}
				return 0;
			}

			// contine process next
			lpszEnd = ::_tcsinc(lpszEnd);
		} while(true);
	}

	
# 37. no member named 'int64_t' in the global namespace using::int64_t

when compile android,

	7. #include <string>
	8. #include <algorithm>
   
compile the lines above,get the error below:

	Compile++ arm  : butterbase <= basetemplate.cpp
	In file included from jni/../../../../base/basebutter.cpp:7:
	D:/android/ndk_r16b/build//../sources/cxx-stl/llvm-libc++/include\cstdint:156:8: error: no member named 'int64_t' in the global namespace
	using::int64_t;
		~~^
	In file included from jni/../../../../base/basebutter.cpp:8:
	In file included from D:/android/ndk_r16b/build//../sources/cxx-stl/llvm-libc++/include\string:470:
	In file included from D:/android/ndk_r16b/build//../sources/cxx-stl/llvm-libc++/include\string_view:169:
	In file included from D:/android/ndk_r16b/build//../sources/cxx-stl/llvm-libc++/include\__string:56:
	In file included from D:/android/ndk_r16b/build//../sources/cxx-stl/llvm-libc++/include\algorithm:643:
	In file included from D:/android/ndk_r16b/build//../sources/cxx-stl/llvm-libc++/include\memory:658:
	D:/android/ndk_r16b/build//../sources/cxx-stl/llvm-libc++/include\atomic:1873:17: error: use of undeclared identifier 'int64_t'
	typedef atomic< int64_t> atomic_int64_t;
          
## Method:

From

	#include <string>
	#include <algorithm>

change to

	typedef long long                int64_t;
	#include <string>
	#include <algorithm>

# 38. Configure


 ```

./configure --host=arm-linux-androideabi --prefix=/install

./configure --host=aarch64-linux-android --prefix=/install

/configure --prefix=/mingw --enable-win32thread \
--host=x86_64-w64-mingw32 --enable-static --enable-shared 


 ```

 ```
--build=BUILD

指定软件包安装的系统平台。如果没有指定，默认值将是'--host'选项的值。


--host=HOST

指定软件运行的系统平台。如果没有指定。将会运行`config.guess'来检测。


--target=GARGET

指定软件面向(target to)的系统平台。这主要在程序语言工具如编译器和汇编器上下文中起作用。如果没有指定，默认将使用'--host'选项的值。


--disable-FEATURE

一些软件包可以选择这个选项来提供为大型选项的编译时配置，例如使用Kerberos认证系统或者一个实验性的编译器最优配置。如果默认是提供这些特性，可以使用'--disable-FEATURE'来禁用它，这里'FEATURE'是特性的名字，例如：

$ ./configure --disable-gui


-enable-FEATURE[=ARG]

相反的，一些软件包可能提供了一些默认被禁止的特性,可以使用'--enable-FEATURE'来起用它。这里'FEATURE'是特性的名字。一个特性可能会接受一个可选的参数。例如：

$ ./configure --enable-buffers=128

`--enable-FEATURE=no'与上面提到的'--disable-FEATURE'是同义的。


--with-PACKAGE[=ARG]

在自由软件社区里，有使用已有软件包和库的优秀传统。当用'configure'来配置一个源码树时，可以提供其他已经安装的软件包的信息。例如，倚赖于Tcl和Tk的BLT器件工具包。要配置BLT，可能需要给'configure'提供一些关于我们把Tcl和Tk装的何处的信息：

 ```


# 39. Rotate Marker

 ```

 // double x, y    the point before rotate;
 // double dx, dy  the base point rotate around;
 // double angle   roate angle in degree;
 // double xO, yO  the point after rotate;

void RotateMarker(double x, double y, double dx, double dy, double angle, double& xO, double& yO)
{
	double dRad = angle*PI / 180.0;
	double dCos = cos(dRad);
	double dSin = sin(dRad);
	xO = x*dCos - y*dSin + (1 - dCos)*dx + dSin*dy;
	yO = x*dSin + y*dCos + (1 - dCos)*dy - dSin*dx;
}

 ```

 # 40. CInternetSession 12057 不能连接到吊销服务器，或者未能获得最终响应 

### ERROR_WINHTTP_SECURE_CERT_REV_FAILED [12057]

```
ERROR_WINHTTP_SECURE_CERT_REV_FAILED
12057
Indicates that revocation cannot be checked because the revocation server was offline (equivalent to CRYPT_E_REVOCATION_OFFLINE).
```
use the following function
**CInternetSession::OpenURL()**;

*CInternetSession* is a part of the Microsoft Foundation Classes C++ library.

some options is related with 'Internet Explore';

compare with windows 10 , windows server has different options;


**Method:**

1. open Internet Explore,find Tools button
2. locate 'Advandced Tab' in Internet options Window;
3. in node 'Security Options', unselect 'Check for server certificate revocation'
   
```
打开 IE->Tools->Internet Options->Advanced Tab->Security Options->”Check for server certificate revocation(Requires Restart)”
```

![Internet Security Options](./img/40-internet_options_security.png)

这个选项当前为选中状态（IE8默认是选中的）。取消这个选项，Bug 症状消失。经过测试发现只有 Windows Server 2003 Standard Edition 的 IE 默认选中该项，而且在其他平台即时此选项选中亦不会发生 Error 12057 (Microsoft’s Bug ? or with other options?)。决定通过编码解决这个问题。
其实编码解决这个问题倒是很简单， 在 HttpOpenRequest 后增加如下代码，设置当前 Http 连接选项取消这个检查

```
DWORD dwFlags = 0;
DWORD dwError = 0;
DWORD dwBuffLen = sizeof(dwFlags);

InternetQueryOption(m_hRequest, INTERNET_OPTION_SECURITY_FLAGS,(LPVOID)&dwFlags, &dwBuffLen);

dwFlags |= SECURITY_FLAG_IGNORE_REVOCATION;

InternetSetOption(m_hRequest, INTERNET_OPTION_SECURITY_FLAGS, (LPVOID)&dwFlags, sizeof(dwFlags)) ;

```


### How to Request URI

1. with CHttpConnection::OpenRequest
   
```

long Request(string strUri, byte** ppbuf, long& len)
{
	// MSDN：CInternetSession第一个参数标识调用Internet功能的应用程序或实体的名称（例如，“Microsoft Internet Browser”）。
	//如果pstrAgent为NULL（缺省值），则框架将调用全局函数AfxGetAppName，该函数返回包含应用程序名称的以null结尾的字符串。
	// 某些协议使用此字符串来标识服务器应用程序。

	CInternetSession    session("BUTTER");
	CHttpFile*	        pHttpFile = NULL;
	long                ret = 0;
	len = 0;
	try
	{
		Log("OpenURL_ERROR", "pHttpFile: %p , Begin OpenRequest", pHttpFile);

		DWORD dwFlags;
		DWORD dwServerType;
		CString strServer, strObject;
		INTERNET_PORT nport;
		AfxParseURL(strUri.c_str(), dwServerType, strServer, strObject, nport);

		CHttpConnection* pHttpConnect = session.GetHttpConnection(strServer, INTERNET_FLAG_SECURE, nport, NULL, NULL);
		if (pHttpConnect)
		{
			pHttpFile = (CHttpFile*)pHttpConnect->OpenRequest(CHttpConnection::HTTP_VERB_POST, strObject, NULL, 1,
				NULL, NULL,
				INTERNET_FLAG_RELOAD | INTERNET_FLAG_NO_CACHE_WRITE | INTERNET_FLAG_KEEP_CONNECTION |
				INTERNET_FLAG_SECURE | INTERNET_FLAG_IGNORE_CERT_CN_INVALID | INTERNET_FLAG_IGNORE_CERT_DATE_INVALID
				//SECURITY_FLAG_IGNORE_REVOCATION
			);
			if (pHttpFile)
			{
				//get web server option
				pHttpFile->QueryOption(INTERNET_OPTION_SECURITY_FLAGS, dwFlags);
				dwFlags |= SECURITY_FLAG_IGNORE_UNKNOWN_CA;  //忽略错误与未知的证书颁发机构
				dwFlags |= SECURITY_FLAG_IGNORE_REVOCATION;  //忽略错误与撤销相关证书
																//set web server option
				pHttpFile->SetOption(INTERNET_OPTION_SECURITY_FLAGS, dwFlags);
				if (pHttpFile->SendRequest())
				{
					//get response status if success, return 200
					DWORD dwStatus = 0;
					DWORD dwStatusLen = sizeof(dwStatus);
					pHttpFile->QueryInfo(HTTP_QUERY_FLAG_NUMBER | HTTP_QUERY_STATUS_CODE, &dwStatus, &dwStatusLen, 0);

					Log("URL_Query", "STATUS_CODE : %d ", dwStatus);

					if (dwStatus == 200 )
					{
						QByteArray buffer;
						char tmpBuf[1024];
						UINT nReadLen = 0;
						while (nReadLen = pHttpFile->Read((void*)tmpBuf, 1024))
						{
							buffer.append(tmpBuf, nReadLen);
						}
						pHttpFile->Close();
						delete pHttpFile;

						if (!buffer.isEmpty())
						{
							len = buffer.length();
							*ppbuf = new byte[len];
							memcpy(*ppbuf, buffer.data(), len);
						}
						ret = 1;
					}
					else
					    ret = -1;
				}else{
				    ret = -2;
				}
			}
			else{
				ret = -3;
			}
		}else
			ret = -4;
	}
	catch (CInternetException* e)
	{
		char szMsg[MAX_PATH] = { "/0" };
		e->GetErrorMessage(szMsg, MAX_PATH);
		Log("URL_ERROR", "error: [%d , %s ; pHttpFile: %p , url:  %s ] ", e->m_dwError, szMsg, pHttpFile,strUri.c_str());

		e->Delete();

		if (pHttpFile)
		{
			pHttpFile->Close();
			delete pHttpFile;
			pHttpFile = NULL;
		}
	}

	if (!pHttpFile)
		Log("URL_ERROR", "pHttpFile: %p ", pHttpFile);

	session.Close();
	return ret;
}

```

2. with CInternetSession::OpenURL

```

long Request(string strUri, byte** ppbuf, long& len)
{
	len = 0;
	CInternetSession    session("BUTTER");
	CHttpFile*	        pHttpFile = NULL;
	long                ret = 0;

	try
	{
		LPCTSTR	 pheader = "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.63 Safari/537.36";
		session.SetOption(INTERNET_OPTION_USER_AGENT, (LPVOID)pheader, strlen(pheader));
		session.SetOption(INTERNET_OPTION_CONNECT_TIMEOUT, 5000);
	    
		DWORD dwFlag = INTERNET_FLAG_TRANSFER_BINARY | INTERNET_FLAG_DONT_CACHE | INTERNET_FLAG_RELOAD;

		pHttpFile = (CHttpFile*)session.OpenURL(strUri.c_str(), 1, dwFlag, pheader);
		if(pHttpFile)
		{
			//get response status if success, return 200
			DWORD dwStatus = 0;
			DWORD dwStatusLen = sizeof(dwStatus);
			pHttpFile->QueryInfo(HTTP_QUERY_FLAG_NUMBER | HTTP_QUERY_STATUS_CODE, &dwStatus, &dwStatusLen, 0);

			Log("URL_Query", "STATUS_CODE : %d ", dwStatus);

			if (dwStatus == 200 )
			{
				QByteArray buffer;
				char tmpBuf[1024];
				UINT nReadLen = 0;
				while (nReadLen = pHttpFile->Read((void*)tmpBuf, 1024))
				{
					buffer.append(tmpBuf, nReadLen);
				}
				pHttpFile->Close();
				delete pHttpFile;

				if (!buffer.isEmpty())
				{
					len = buffer.length();
					*ppbuf = new byte[len];
					memcpy(*ppbuf, buffer.data(), len);
				}
				ret = 1;
			}
		}
		else
		    ret = -1;
		
	}
	catch (CInternetException* e)
	{
		char szMsg[MAX_PATH] = { "/0" };
		e->GetErrorMessage(szMsg, MAX_PATH);
		Log("URL_ERROR", "error: [%d , %s ; pHttpFile: %p , url:  %s ] ", e->m_dwError, szMsg, pHttpFile, strUri.c_str());

		e->Delete();
		if (pHttpFile)
		{
			pHttpFile->Close();
			delete pHttpFile;
			pHttpFile = NULL;
		}
	}

	if (!pHttpFile)
	{
		Log("URL_ERROR", "pHttpFile: %p ", pHttpFile);
		ret = -2;
	}
	
	session.Close();
	return ret;
}

```

 # 41. Mapbox satelite url token


 1. visit url http://www.mapbox.cn/mapbox-gl-js/example/satellite-map/ ，
 2. from the example,use F12 --> NetWork --> get the request url :
    https://api.mapbox.com/v4/mapbox.satellite/14/13402/6763.webp?sku=1015TlSrLHf3s&access_token=pk.eyJ1IjoiZXhhbXBsZXMiLCJhIjoiY2p1dHRybDR5MGJuZjQzcGhrZ2doeGgwNyJ9.a-vxW4UaxOoUMWUTGnEArw
 3. from the url above,get the token "pk.eyJ1IjoiZXhhbXBsZXMiLCJhIjoiY2p1dHRybDR5MGJuZjQzcGhrZ2doeGgwNyJ9.a-vxW4UaxOoUMWUTGnEArw";

## vector tile
https://docs.mapbox.com/api/maps/vector-tiles/

https://api.mapbox.com/v4/{tileset_id}/{zoom}/{x}/{y}.{format}

https://api.mapbox.com/v4/mapbox.mapbox-streets-v8/12/1171/1566.mvt?style=mapbox://styles/mapbox/streets-v11@00&access_token=YOUR_MAPBOX_ACCESS_TOKEN

## raster tile
https://api.mapbox.com/v4/{tileset_id}/{zoom}/{x}/{y}{@2x}.{format}

https://api.mapbox.com/v4/mapbox.satellite/1/0/0@2x.jpg90?access_token=YOUR_MAPBOX_ACCESS_TOKEN

## template

https://docs.mapbox.com/api/maps/static-tiles/

https://api.mapbox.com/styles/v1/{username}/{style_id}/tiles/{tilesize}/{z}/{x}/{y}{@2x}

Retrieve raster tiles from a Mapbox Studio style.

The returned raster tile will be 512 pixels by 512 pixels by default.
If the queried tileset contains raster layers, the returned tile will be a JPEG.
If the queried tileset contains only vector layers, the returned tile will be a PNG.

* demo

https://api.mapbox.com/styles/v1/mapbox/satellite-v9/tiles/512/2/3/1@2x?access_token=pk.eyJ1IjoiZXhhbXBsZXMiLCJhIjoiY2p1dHRybDR5MGJuZjQzcGhrZ2doeGgwNyJ9.a-vxW4UaxOoUMWUTGnEArw

 ## mapbox demo

```
<!DOCTYPE html>
<html>
<head>
	<meta charset='utf-8' />
	<title>显示卫星地图</title>
	<meta name='viewport' content='initial-scale=1,maximum-scale=1,user-scalable=no' />
	<script src='https://api.tiles.mapbox.com/mapbox-gl-js/v1.1.1/mapbox-gl.js'></script>
	<link href='https://api.tiles.mapbox.com/mapbox-gl-js/v1.1.1/mapbox-gl.css' rel='stylesheet' />
	<style>
		body { margin:0; padding:0; }
		#map { position:absolute; top:0; bottom:0; width:100%; }
	</style>
</head>
<body>

<div id='map'></div>
<script>
mapboxgl.accessToken = 'pk.eyJ1IjoiZXhhbXBsZXMiLCJhIjoiY2p1dHRybDR5MGJuZjQzcGhrZ2doeGgwNyJ9.a-vxW4UaxOoUMWUTGnEArw';
var map = new mapboxgl.Map({
	container: 'map',
	zoom: 9,
	center: [137.9150899566626, 36.25956997955441],
	style: 'mapbox://styles/mapbox/satellite-v9'
});
</script>

</body>
</html>

```

# 42 compile podofo

## 1. steps

```
get tar file 'podofo-0.9.6.tar.gz'

tar zxvf podofo-0.9.6.tar.gz

mkdir buildpdf

cd buildpdf

cmake ../podofo-0.9.6 -DCMAKE_INSTALL_PREFIX=/home/cycle/src/intall -DPODOFO_BUILD_SHARED:BOOL=TRUE -DPODOFO_BUILD_STATIC:BOOL=FALSE -DPNG_LIBRARY_RELEASE:FILEPATH=/home/cycle/sdk/3rd/png/lib/libpng.so -DPNG_PNG_INCLUDE_DIR:PATH=/home/cycle/sdk/3rd/png/include/libpng16

make

make install


```

## 2. compile only with jpeg / png / freetype without OpenSSL and tiff

### 1. modify the CMakeLists.txt;
   located  'FIND_PACKAGE(*libname*)'
   and make it invalid;
   
```
change from 
FIND_PACKAGE(LIBCRYPTO)
FIND_PACKAGE(LIBIDN)
FIND_PACKAGE(TIFF)
FIND_PACKAGE(OpenSSL)
to

#FIND_PACKAGE(LIBCRYPTO)
#FIND_PACKAGE(LIBIDN)
#FIND_PACKAGE(TIFF)
#FIND_PACKAGE(OpenSSL)

```
### 2. change the library path
   
jpeg.so / png.so / freetype.so located in folder '/home/x64/program' which is the prefer folder rather than the default folder such as '/usr/lib64';

```
[core@localhost pdf]$ echo $LD_LIBRARY_PATH
/home/x64/program

[core@localhost pdf]$ export PATH=$LD_LIBRARY_PATH:$PATH
[core@localhost pdf]$ echo $PATH
/home/x64/program:/usr/lib64/ccache:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/core/.local/bin:/home/core/bin
```

### 3. configure and compile 

```
[core@localhost pdf]$   cmake ../podofo-0.9.6 -DCMAKE_INSTALL_PREFIX=/home/cycle/bin/install -DPODOFO_BUILD_SHARED:BOOL=TRUE -DPODOFO_BUILD_STATIC:BOOL=FALSE  -DPODOFO_NO_FONTMANAGER:BOOL=TRUE -DLIBJPEG_INCLUDE_DIR=/home/core/sdk/3rd/jpeg/include_linux_x86_64  -DPNG_PNG_INCLUDE_DIR=/home/core/sdk/3rd/png/include

cmake ../podofo-0.9.6 -DCMAKE_INSTALL_PREFIX=/home/cycle/pdf-shared -DPODOFO_BUILD_SHARED:BOOL=TRUE -DPODOFO_BUILD_STATIC:BOOL=FALSE -DPODOFO_NO_FONTMANAGER:BOOL=TRUE  -DPNG_LIBRARY_RELEASE:FILEPATH=/home/cycle/program/libpng16.so -DPNG_PNG_INCLUDE_DIR:PATH=/home/cycle/3rd/png/include -DLIBJPEG_LIBRARY_RELEASE:FILEPATH=/home/cycle/program/libjpeg.so -DLIBJPEG_INCLUDE_DIR:PATH=/home/cycle/3rd/jpeg/include_linux_x86_64 -DFREETYPE_INCLUDE_DIR_FT2BUILD:PATH=/home/cycle/3rd/freetype/include/freetype2 -DFREETYPE_INCLUDE_DIR_FTHEADER:PATH=/home/cycle/3rd/freetype/include/freetype2 -DFREETYPE_LIBRARY_RELEASE:FILEPATH=/home/cycle/3rd/freetype/lib/libfreetype.so 

-- The C compiler identification is GNU 4.9.3
-- The CXX compiler identification is GNU 4.9.3
-- Check for working C compiler: /usr/lib64/ccache/cc
-- Check for working C compiler: /usr/lib64/ccache/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/lib64/ccache/c++
-- Check for working CXX compiler: /usr/lib64/ccache/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
WANT_LIB64 unset; assuming normal library directory names
Will install libraries to /home/cycle/bin/install/lib
-- Looking for strings.h
-- Looking for strings.h - found
-- Looking for arpa/inet.h
-- Looking for arpa/inet.h - found
-- Looking for winsock2.h
-- Looking for winsock2.h - not found
-- Looking for mem.h
-- Looking for mem.h - not found
-- Looking for ctype.h
-- Looking for ctype.h - found
-- Looking for sys/types.h
-- Looking for sys/types.h - found
-- Looking for stdint.h
-- Looking for stdint.h - found
-- Looking for BaseTsd.h
-- Looking for BaseTsd.h - not found
-- Looking for sys/types.h
-- Looking for sys/types.h - found
-- Looking for stdint.h
-- Looking for stdint.h - found
-- Looking for stddef.h
-- Looking for stddef.h - found
-- Check size of long int
-- Check size of long int - done
-- Check size of int64_t
-- Check size of int64_t - done
-- Check if the system is big endian
-- Searching 16 bit integer
-- Check size of unsigned short
-- Check size of unsigned short - done
-- Using unsigned short
-- Check if the system is big endian - little endian
Using gcc specific compiler options
-- Found ZLIB: /home/x64/program/libz.so  
Found zlib headers in /usr/local/include, library at /home/x64/program/libz.so
OpenSSL's libCrypto not found. Encryption support will be disabled
Libidn not found. AES-256 Encryption support will be disabled
-- Found LIBJPEG: /home/x64/program/libjpeg.so  
Found libjpeg headers in /home/core/sdk/3rd/jpeg/include_linux_x86_64, library at /home/x64/program/libjpeg.so
Libtiff not found. TIFF support will be disabled
-- Found ZLIB: /home/x64/program/libz.so (found version "1.2.3") 
-- Found PNG: /home/x64/program/libpng.so (found version "1.6.38.git") 
Found LibPng headers in /home/core/sdk/3rd/png/include;/usr/local/include, library at /home/x64/program/libpng.so;/home/x64/program/libz.so
-- Could NOT find UNISTRING (missing: UNISTRING_INCLUDE_DIR UNISTRING_LIBRARY) 
LibUnistring not found. Unistring support will be disabled
-- Ensure you cppunit installed version is at least 1.12.0
Cppunit not found. No unit tests will be built.
Found freetype library at /usr/lib64/libfreetype.so, headers /usr/include/freetype2
-- Found PkgConfig: /usr/bin/pkg-config (found version "0.28") 
-- Checking for module 'fontconfig'
--   Found fontconfig, version 2.11.1
-- Found Fontconfig: fontconfig;freetype  
-- Could NOT find Lua50 (missing: LUA_LIBRARIES LUA_INCLUDE_DIR) 
-- Could NOT find Lua (missing: LUA_LIBRARIES LUA_INCLUDE_DIR) 
Lua not found - PoDoFoImpose and PoDoFoColor will be built without Lua support
Building multithreaded version of PoDoFo.
-- Looking for pthread.h
-- Looking for pthread.h - found
-- Looking for pthread_create
-- Looking for pthread_create - not found
-- Looking for pthread_create in pthreads
-- Looking for pthread_create in pthreads - not found
-- Looking for pthread_create in pthread
-- Looking for pthread_create in pthread - found
-- Found Threads: TRUE  
Building shared PoDoFo library
Pkg-config found, creating a pkg-config file for linking against shared library.
-- Configuring done
-- Generating done
-- Build files have been written to: /home/core/tmp/5.12/pdf


```

### 4. compile and install
   
```
make install
```

### 5. check it

```
[root@localhost program]# ldd libpodofo.so
ldd: warning: you do not have execution permission for `./libpodofo.so'
linux-vdso.so.1 =>  (0x00007ffdf2174000)
libz.so.1 => /home/x64/program/libz.so.1 (0x00007fbc8ec80000)
libjpeg.so.9 => /home/x64/program/libjpeg.so.9 (0x00007fbc8ea27000)
libpthread.so.0 => /lib64/libpthread.so.0 (0x00007fbc8e80b000)
libfreetype.so.6 => /home/x64/program/libfreetype.so.6 (0x00007fbc8e55b000)
libpng16.so.16 => /home/x64/program/libpng16.so.16 (0x00007fbc8e313000)
libstdc++.so.6 => /lib64/libstdc++.so.6 (0x00007fbc8e00c000)
libm.so.6 => /lib64/libm.so.6 (0x00007fbc8dd0a000)
libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00007fbc8daf4000)
libc.so.6 => /lib64/libc.so.6 (0x00007fbc8d727000)
/lib64/ld-linux-x86-64.so.2 (0x00007fbc8f312000)

```


# 43  brush rotate

![pdfpainter Brush rotate angle](./img/43-pdfpainter_brush_rotate_angle.png)

```
PdfPainter m_painter;
void  DrawRect(double dStartX, double dStartY, double dEndX, double dEndY)
{
	m_painter.DrawLine(dStartX, dStartY, dStartX, dEndY);
	m_painter.DrawLine(dStartX, dEndY, dEndX, dEndY);
	m_painter.DrawLine(dEndX, dEndY, dEndX, dStartY);
	m_painter.DrawLine(dEndX, dStartY, dStartX, dStartY);

	m_painter.DrawLine(dStartX, dStartY, dEndX, dEndY);
	m_painter.DrawLine(dStartX, dEndY, dEndX, dStartY);
}

void  Fill(bool useEvenOddRule)
{
	PdfXObject* m_pXPattern = new PdfXObject(PdfRect(0, 0, patWid,patHei), new PdfMemDocument());

	if (m_bBeginPattern) {
		PdfPainter painter;
		painter.SetPage(m_pXPattern);
		painter.Fill();
		painter.FinishPage();
	}
	else {
		if (m_pXPattern != NULL)
		{
			m_painter.Save();
			m_painter.ClosePath();
			m_painter.Clip(true);

			QBrush *pBrush;
			D_RECT  rc = {0.0}; 
			memcpy(&rc, &m_pathRect, sizeof(D_RECT));
			double dOrigin = 0.0;
			double xScale, yScale; xScale = yScale = 1.0;
			double xStep = m_pXPattern->GetPageSize().GetWidth()*xScale, yStep = m_pXPattern->GetPageSize().GetHeight()*yScale;
			
			QTransform  trans;
			if (pBrush)
				trans = pBrush->transform();

			QRectF rect = trans.mapRect(QRectF(rc.xmin, rc.ymax, rc.xmax - rc.xmin, rc.ymax - rc.ymin));
			QPointF center = rect.center();
			QPointF  bl = rect.bottomLeft();
			QPointF  tr = rect.topRight();

			double xBound = rc.xmax - rc.xmin;
			double yBound = rc.ymax - rc.ymin;

			m_painter.DrawXObject((rc.xmax + rc.xmin) / 2, (rc.ymax + rc.ymin) / 2, m_pXPattern, xScale, yScale);

			m_painter.SetTransformationMatrix(trans.m11(), trans.m21(), trans.m12(), trans.m22(), 0, 0);

			DrawRect(bl.rx() + 1.5 - yBound, bl.ry() + 1.5, tr.rx() - 1.5 - yBound, tr.ry() - 1.5);

			//m_painter.DrawLine(center.rx(), center.ry(), (rc.xmax + rc.xmin) / 2.0, (rc.ymax + rc.ymin) / 2.0);
			m_painter.DrawLine(center.rx(), center.ry(), 0, 0);

			//	DrawRect(0, 0, xStep, yStep);
			m_painter.DrawLine(0, 0, xStep, yStep);

			m_painter.DrawXObject(center.rx(), center.ry(), m_pXPattern, xScale, yScale);

			m_painter.DrawXObject(center.rx() - yBound, center.ry(), m_pXPattern, xScale, yScale);
			m_painter.DrawXObject(center.rx() - yBound - xStep, center.ry(), m_pXPattern, xScale, yScale);
			m_painter.DrawXObject(center.rx() - yBound + xStep, center.ry(), m_pXPattern, xScale, yScale);

			double xmin = bl.rx() - yBound;
			double xmax = tr.rx() - yBound;
			double ymin = min(bl.ry(), tr.ry());
			double ymax = max(bl.ry(), tr.ry());

			int nXmin = floor((xmin - dOrigin) / xStep);
			int nXmax = ceil((xmax - dOrigin) / xStep);
			int nYmin = floor((ymin - dOrigin) / yStep);
			int nYmax = ceil((ymax - dOrigin) / yStep);

			for (int i = nXmin; i <= nXmax; ++i)
			{
				for (int j = nYmin; j <= nYmax; ++j)
				{
					DOT tmpDot = {};
					tmpDot.x = dOrigin + i * xStep;
					tmpDot.y = dOrigin + j * yStep;
					m_painter.DrawXObject(tmpDot.x, tmpDot.y, m_pXPattern, xScale, yScale);
				}
			}

			m_painter.Restore();
		}
		else
		{
			m_painter.Fill();
		}
		DELETEPOINTER(m_pXPattern);
	}
}

```

# 44 . Matrix to angle

```
/**************Matrix***********
 *     | m11     m12     0 |
 *     | m21     m22     0 |
 *     | 0       0       1 |
 ************************/

double ComputeAngleByMatix()
{
	double angle = 0;
	double m11 = m_matrix.m11();
	double m21 = m_matrix.m21();
	double m12 = m_matrix.m12();
	double m22 = m_matrix.m22();
	float sy = sqrt(m11*m11 + m21*m21);
	bool singular = sy < 1e-6;
	if (!singular)
		angle = atan2(m21, m11);
	else 
		angle = 0;

	double dAn = angle * 180 / PI;
	return dAn;
}
```

# 45 .  Rotate the point around the center point

![Rotate the point around the center point](./img/45-rotate_angle_refer_line.png)
 
* Note the refer line ;
  
  
```
bool RotateDot(double &dotX, double &dotY, double centerX, double centerY, double dAngle)
{
	double		x = 0.0, y = 0.0;
	x = dotX - centerX;
	y = dotY - centerY;
	dotX = centerX + x*cos(dAngle) - y*sin(dAngle);
	dotY = centerY + x*sin(dAngle) + y*cos(dAngle);
	return true;
}
```

# 46.  unknow function in c sharp setttings 

VS2019调试模式， 在调试的过程中,表达式计算器中发生内部错误，c#编译器内部错误 调试堆栈显示  未知函数

解决办法：

	工具--选项--调试--常规，将“使用托管兼容模式”勾选中即可，如下图所示：

![c sharp setttings ](./img/46-c_sharp_settings.png)
 
# 47.  3d model url 

Gltf/obj模型下载

https://github.com/KhronosGroup/glTF

https://playcanvas.com/
https://playcanvas.com/viewer

https://www.kenney.nl/

https://sketchfab.com/

# 48. wuhan tong ji nian jian

http://tjj.wuhan.gov.cn/tjfw/tjnj/


# 49. compile Freetype

1. get the source code 'freetype-2.12.1.tar'
2. locate the folder of the freetype tar file ,eg: /home/ft
   cd /home/ft
3. unzip the tar file; 
   tar xvf freetype-2.12.1.tar
4. open root folder of freetype
   cd /home/ft/freetype-2.12.1
5. ./configure --prefix=/home/ft/ft-static --enable-static --with-png=no
6. if with png,configure '--with-png=yes';
   perhaps compiler complain that can not find 'png.h';
   for this case,set LIBPNG_CFLAGS and LIBPNG_LIBS;

   ./configure --prefix=/home/ft/ft-shared --enable-shared LIBPNG_CFLAGS='-I/home/png/include' LIBPNG_LIBS='-L/home/program -lpng16' 

7. make install
8. locate folder '/home/ft/ft-static',get the compiled library and header file;


# 50. fPIC

1、-fPIC 作用于编译阶段，在编译动态库时(.so文件)告诉编译器产生与位置无关代码(Position-Independent Code)，若未指定-fPIC选项编译.so文件，则在加载动态库时需进行重定向。

2、64位编译器下编译生成动态库时，出现以下错误：

/usr/lib64/gcc/x86_64-suse-linux/4.3/../../../../x86_64-suse-linux/bin/ld: ../../CI/script/server/lib/libz.a(adler32.o): relocation R_X86_64_32 against `.text' can not be used when making a shared object; recompile with -fPIC

../../CI/script/server/lib/libz.a: could not read symbols: Bad value

原因：提示说需要-fPIC编译，然后在链接动态库的地方加上-fPIC的参数编译结果还是报错，需要把共享库所用到的所有静态库都采用-fPIC编译一遍，才可以成功的在64位环境下编译出动态库。


    add_compile_options(-fPIC)
	
```
/usr/bin/ld: /home/core2/master/sdk/3rd/podofo/linux_x86_64/libpodofo.a(PdfArray.cpp.o): relocation R_X86_64_32S against symbol `_ZTVN6PoDoFo8PdfArrayE' can not be used when making a shared object; recompile with -fPIC
/usr/bin/ld: /home/core2/master/sdk/3rd/podofo/linux_x86_64/libpodofo.a(PdfCanvas.cpp.o): relocation R_X86_64_32 against `.rodata' can not be used when making a shared object; recompile with -fPIC
/usr/bin/ld: /home/core2/master/sdk/3rd/podofo/linux_x86_64/libpodofo.a(PdfColor.cpp.o): relocation R_X86_64_32S against symbol `_ZTVN6PoDoFo8PdfColorE' can not be used when making a shared object; recompile with -fPIC
/usr/bin/ld: /home/core2/master/sdk/3rd/podofo/linux_x86_64/libpodofo.a(PdfDataType.cpp.o): relocation R_X86_64_32S against symbol `_ZTVN6PoDoFo11PdfDataTypeE' can not be used when making a shared object; recompile with -fPIC
/usr/bin/ld: /home/core2/master/sdk/3rd/podofo/linux_x86_64/libpodofo.a(PdfDictionary.cpp.o): relocation R_X86_64_32S against symbol `_ZTVN6PoDoFo13PdfDictionaryE' can not be used when making a shared object; recompile with -fPIC
/usr/bin/ld: /home/core2/master/sdk/3rd/podofo/linux_x86_64/libpodofo.a(PdfEncodingFactory.cpp.o): relocation R_X86_64_32 against symbol `_ZN6PoDoFo18PdfEncodingFactory7s_mutexE' can not be used when making a shared object; recompile with -fPIC
```

# 51. build podofo with static freetype library which is independent with png lib

  case: podofo lib depend on png 、jpeg 、freetype, besides this, freetype library also depend on png, they may conflict with each other because of the unmatched version;
```
  podofo
    |-----png
	|-----jpeg
	|-----freetype
	        |------png
			|------bz
			
```
 in linux, freetype usually can be located and found in the system path,we cannot remove it because the OS use it; if we wanna promote the version,we must have the root authority;
 for somewhere,we may just need link  freetype which has the special version, the same file in system path perhaps is not what we need;

 so we assemble the static freetype into the target library,here is podofo;

 ## 1. compile and build the static freetype
   
```
	../freetype-2.12.1/configure --prefix=/home/cycle/install/static --enable-static --enable-shared=no --with-png=no --with-bzip2=no --with-pic
```
* --enable-static --enable-shared=no
  build static library

* --with-bzip2=no
  static freetype need static bzip,if donnot have static bzip,ignore it;

* --with-pic
  when making a shared object,should compile with -fPIC

	/usr/bin/ld: /home/bin/podofo/libpodofo.a(PdfColor.cpp.o): relocation R_X86_64_32S against symbol `_ZTVN6PoDoFo8PdfColorE' can not be used when making a shared object; recompile with -fPIC

## 2. compile podofo with the static freetype
   
```
    mkfir pdf
    cd pdf
    cmake ../podofo-0.9.6 -DCMAKE_INSTALL_PREFIX=/home/cycle/install/pdf/share  -DPODOFO_BUILD_SHARED:BOOL=TRUE -DPODOFO_BUILD_STATIC:BOOL=FALSE -DPODOFO_NO_FONTMANAGER:BOOL=TRUE  -DPNG_LIBRARY_RELEASE:FILEPATH=/home/cycle/bin/libpng16.so -DPNG_PNG_INCLUDE_DIR:PATH=/home/cycle/sdk/3rd/png/include -DLIBJPEG_LIBRARY_RELEASE:FILEPATH=/home/cycle/bin/libjpeg.so -DLIBJPEG_INCLUDE_DIR:PATH=/home/cycle/sdk/3rd/jpeg/include_linux_x86_64 -DFREETYPE_INCLUDE_DIR_FT2BUILD:PATH=/home/cycle/install/static/include/freetype2 -DFREETYPE_INCLUDE_DIR_FTHEADER:PATH=/home/cycle/install/static/include/freetype2 -DFREETYPE_LIBRARY_RELEASE:FILEPATH=/home/cycle/install/static/lib/libfreetype.a 
	make 
	make install
```
## 3. check the target lib
    
BEFORE
```
[cycle@localhost p]$ ldd /home/cycle/bin/libpodofo.so
libz.so.1 => /home/cycle/bin/libz.so.1 (0x000000fff64a8000)
libjpeg.so.9 => /home/cycle/bin/libjpeg.so.9 (0x000000fff6424000)
libpthread.so.0 => /lib64/libpthread.so.0 (0x000000fff63bc000)
libfreetype.so.6 => /lib64/libfreetype.so.6 (0x000000fff62f0000)
libpng16.so.16 => /home/cycle/bin/libpng16.so.16 (0x000000fff6288000)
libstdc++.so.6 => /lib64/libstdc++.so.6 (0x000000fff611c000)
libm.so.6 => /lib64/libm.so.6 (0x000000fff6020000)
libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x000000fff5fbc000)
libc.so.6 => /lib64/libc.so.6 (0x000000fff5dd0000)
/lib64/ld.so.1 (0x000000fff6818000)
libbz2.so.1 => /lib64/libbz2.so.1 (0x000000fff5dac000)
```

AFTER

```
[cycle@localhost p]$ ldd /home/cycle/bin/libpodofo.so
libz.so.1 => /home/cycle/bin/libz.so.1 (0x000000ffea934000)
libjpeg.so.9 => /home/cycle/bin/libjpeg.so.9 (0x000000ffea8b0000)
libpthread.so.0 => /lib64/libpthread.so.0 (0x000000ffea848000)
libpng16.so.16 => /home/cycle/bin/libpng16.so.16 (0x000000ffea7e0000)
libstdc++.so.6 => /lib64/libstdc++.so.6 (0x000000ffea674000)
libm.so.6 => /lib64/libm.so.6 (0x000000ffea578000)
libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x000000ffea514000)
libc.so.6 => /lib64/libc.so.6 (0x000000ffea328000)
/lib64/ld.so.1 (0x000000ffead44000)
```

# 52. JNLP

```
Microsoft Windows [版本 6.1.7601]
版权所有 (c) 2009 Microsoft Corporation。保留所有权利。

C:\Users\Administrator>keytool -v
用法错误: 没有提供命令
密钥和证书管理工具

命令:

 -certreq            生成证书请求
 -changealias        更改条目的别名
 -delete             删除条目
 -exportcert         导出证书
 -genkeypair         生成密钥对
 -genseckey          生成密钥
 -gencert            根据证书请求生成证书
 -importcert         导入证书或证书链
 -importpass         导入口令
 -importkeystore     从其他密钥库导入一个或所有条目
 -keypasswd          更改条目的密钥口令
 -list               列出密钥库中的条目
 -printcert          打印证书内容
 -printcertreq       打印证书请求的内容
 -printcrl           打印 CRL 文件的内容
 -storepasswd        更改密钥库的存储口令

使用 "keytool -command_name -help" 获取 command_name 的用法

C:\Users\Administrator>F:

F:\>cd F:\program\web\tomcat-10.0.21\webapps\jnlp

F:\program\web\tomcat-10.0.21\webapps\jnlp>keytool -genkey -keystore keycookies
-alias kcookies
输入密钥库口令:
密钥库口令太短 - 至少必须为 6 个字符
输入密钥库口令:
再次输入新口令:
您的名字与姓氏是什么?
  [Unknown]:  cycle
您的组织单位名称是什么?
  [Unknown]:  cycle
您的组织名称是什么?
  [Unknown]:  cycle
您所在的城市或区域名称是什么?
  [Unknown]:  wh
您所在的省/市/自治区名称是什么?
  [Unknown]:  HN
该单位的双字母国家/地区代码是什么?
  [Unknown]:  CHN
CN=cycle, OU=cycle, O=cycle, L=wh, ST=HN, C=CHN是否正确?
  [否]:  y

输入 <kcookies> 的密钥口令
        (如果和密钥库口令相同, 按回车):
再次输入新口令:

Warning:
JKS 密钥库使用专用格式。建议使用 "keytool -importkeystore -srckeystore keycookie
s -destkeystore keycookies -deststoretype pkcs12" 迁移到行业标准格式 PKCS12。

F:\program\web\tomcat-10.0.21\webapps\jnlp>keytool -list -keystore keycookies
输入密钥库口令:
密钥库类型: JKS
密钥库提供方: SUN

您的密钥库包含 1 个条目

kcookies, 2022-7-13, PrivateKeyEntry,
证书指纹 (SHA1): F0:FA:A9:1E:60:E6:81:43:37:45:0C:37:DD:8A:62:E2:41:71:CC:75

Warning:
JKS 密钥库使用专用格式。建议使用 "keytool -importkeystore -srckeystore keycookie
s -destkeystore keycookies -deststoretype pkcs12" 迁移到行业标准格式 PKCS12。

F:\program\web\tomcat-10.0.21\webapps\jnlp>jarsigner -keystore keycookies butter
_cookies-1.5-6.jar kcookies
输入密钥库的密码短语:
jar 已签名。

警告:
签名者证书将在六个月内过期。
未提供 -tsa 或 -tsacert, 此 jar 没有时间戳。如果没有时间戳, 则在签名者证书的到期
日期 (2022-10-11) 或以后的任何撤销日期之后, 用户可能无法验证此 jar。

F:\program\web\tomcat-10.0.21\webapps\jnlp>jarsigner -keystore keycookies butter
_ribbon-1.2-6.jar kcookies
输入密钥库的密码短语:
jar 已签名。

警告:
签名者证书将在六个月内过期。
未提供 -tsa 或 -tsacert, 此 jar 没有时间戳。如果没有时间戳, 则在签名者证书的到期
日期 (2022-10-11) 或以后的任何撤销日期之后, 用户可能无法验证此 jar。

F:\program\web\tomcat-10.0.21\webapps\jnlp>jarsigner -keystore keycookies butter
_base-1.0.1-6.jar kcookiess
输入密钥库的密码短语:
jarsigner: 找不到kcookiess的证书链。kcookiess必须引用包含私有密钥和相应的公共密
钥证书链的有效密钥库密钥条目。

F:\program\web\tomcat-10.0.21\webapps\jnlp>jarsigner -keystore keycookies butter
_jni-1.5-6.jar kcookies
输入密钥库的密码短语:
jar 已签名。

F:\program\web\tomcat-10.0.21\webapps\jnlp>

```


# 53. publish java web to tomcat server

1. prepare tomcat server
    * visit the official website: https://tomcat.apache.org/ ;
	* download the tomcat install package corresponding with your OS;
	* unzip the package into folder such as '../tomcat-10.0.21',
	  in the folder \tomcat-10.0.21\bin ,find the bat file 'startup.bat'
	* click the bat file , then in chrome explore ,visit url 'http://localhost:8080/'
      if find some information about tomcat ,that show tomcat is prepared OK;

2. create java web project
   * open IntelliJ IDEA,create 'new Module' with 'Maven'
   * 'Create form archetype' select 'maven-archetype-webapp',press 'NEXT'
   * input 'GroupId' 'ArctifactId' 'Version' ,press 'NEXT'
   * select Maven Directory,press 'NEXT'
   * then configure the Module above
   * in 'Project Structure',' Project Settings' --> 'Facets',add web facet will be added to the selected Module above;

3. package web project into '*.war'
   * in Maven tree,select the target module,compile and package
   * we can get the target war file 'moudle-name.war' in folder '/moudle-name/target/';


4. publish and check
   * copy the target war file 'moudle-name.war' into tomcat folder 'tomcat-10.0.21\webapps '

5. check and visit web service
   * start tomcat, then visit url 'http://localhost:8080/module-name/'
   * we get some information such as 'Hello world!'


# 54. Price list

10,000/100 m^2

```
1,A1,182395,112950,2017-2020,61.92603964
2,A1-A3,275233,117228,2012-2014,42.59227636
3,B,174450,66686,2018-2021,38.22642591
4,NANSHANF,144767,36202,2020-2022,25.00708034
5,Hope,196530,46217,2020-2022,23.51651147
6,1-033,173070,26133,2021-2023,15.09967065
7,ELake-star,179282,86555,2021-2025,48.27868944
8,KT1,235182,63179,2019-2022,26.86387564
9,KT2,100339,23210,2019-2021,23.13158393
10,PEAK,130952,50339,2022-2025,38.44080274
11,ZRHR,69158,15079,2021-2022,21.80369588
12,ZR,140887,22342,2020-2021,15.85809904
13,PENGHUWan,71502,13656,2021-2023,19.09876647
14,DaGONGGuan,353460,99991,2022-2024,28.28919821
15,DAJIA,97804,13577,202112-202406,13.88184532
16,DZ,64627,21396,202107-202401,33.10690578
```

# 55. base64 encode and decode

```
std::string const base64_chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";

class helper
{

public:
	helper() {}
	~helper() {}

	template<class T>
	static std::string vector_join(const std::vector<T>& v, const std::string& token, const bool useQuotes = false) {
		std::ostringstream result;
		for (typename std::vector<T>::const_iterator i = v.begin(); i != v.end(); i++) {
			if (i != v.begin()) result << token;
			if (useQuotes)
				result << "\"" << *i << "\"";
			else
				result << *i;
		}
		return result.str();
	}

	static std::string JsonEscape(const std::string &source) {
		std::string result;
		for (const char* c = source.c_str(); *c != 0; ++c) {
			switch (*c) {
			case '\"':
				result += "\\\"";
				break;
			case '\\':
				result += "\\\\";
				break;
			case '\b':
				result += "\\b";
				break;
			case '\f':
				result += "\\f";
				break;
			case '\n':
				result += "\\n";
				break;
			case '\r':
				result += "\\r";
				break;
			case '\t':
				result += "\\t";
				break;
			default:
				if (iscntrl(*c)) {
					std::ostringstream oss;
					oss << "\\u" << std::hex << std::uppercase << std::setfill('0')
						<< std::setw(4) << static_cast<int>(*c);
					result += oss.str();
				}
				else {
					result += *c;
				}
				break;
			}
		}
		return result;
	}

	static unsigned short GetItem2Byte(const unsigned char* data, unsigned int index) {
		return (data[index + 1] << 8) | data[index + 0];
	}

	static unsigned int GetItem4Byte(const unsigned char* data, unsigned int index) {
		return
			(((unsigned long long)data[index + 3]) << 32) |
			(((unsigned int)data[index + 2]) << 16) |
			(((unsigned int)data[index + 1]) << 8) |
			data[index + 0];
	}

	static std::string GetItemString(const unsigned char* data, unsigned int index, unsigned int length) {
		std::string str((char*)&data[index], (size_t)length);
		return str;
	}

	static std::wstring GetItemStringW(const unsigned char* data, unsigned int index, unsigned int length) {
		std::wstring str(reinterpret_cast<wchar_t*>(data[index]), (size_t)length / sizeof(wchar_t));
		return str;
	}

	static inline bool is_base64(unsigned char c) {
		return (isalnum(c) || (c == '+') || (c == '/'));
	}

	static std::string base64_encode(unsigned char const* bytes_to_encode, unsigned int in_len) {
		std::string ret;
		int i = 0;
		int j = 0;
		unsigned char char_array_3[3];
		unsigned char char_array_4[4];

		while (in_len--) {
			char_array_3[i++] = *(bytes_to_encode++);
			if (i == 3) {
				char_array_4[0] = (char_array_3[0] & 0xfc) >> 2;
				char_array_4[1] = ((char_array_3[0] & 0x03) << 4) + ((char_array_3[1] & 0xf0) >> 4);
				char_array_4[2] = ((char_array_3[1] & 0x0f) << 2) + ((char_array_3[2] & 0xc0) >> 6);
				char_array_4[3] = char_array_3[2] & 0x3f;

				for (i = 0; (i < 4); i++)
					ret += base64_chars[char_array_4[i]];
				i = 0;
			}
		}

		if (i)
		{
			for (j = i; j < 3; j++)
				char_array_3[j] = '\0';

			char_array_4[0] = (char_array_3[0] & 0xfc) >> 2;
			char_array_4[1] = ((char_array_3[0] & 0x03) << 4) + ((char_array_3[1] & 0xf0) >> 4);
			char_array_4[2] = ((char_array_3[1] & 0x0f) << 2) + ((char_array_3[2] & 0xc0) >> 6);
			char_array_4[3] = char_array_3[2] & 0x3f;

			for (j = 0; (j < i + 1); j++)
				ret += base64_chars[char_array_4[j]];

			while ((i++ < 3))
				ret += '=';

		}

		return ret;

	}
	static std::string base64_decode(std::string const& encoded_string) {
		size_t in_len = encoded_string.size();
		size_t i = 0;
		size_t j = 0;
		int in_ = 0;
		unsigned char char_array_4[4], char_array_3[3];
		std::string ret;

		while (in_len-- && (encoded_string[in_] != '=') && is_base64(encoded_string[in_])) {
			char_array_4[i++] = encoded_string[in_]; in_++;
			if (i == 4) {
				for (i = 0; i < 4; i++)
					char_array_4[i] = static_cast<unsigned char>(base64_chars.find(char_array_4[i]));

				char_array_3[0] = (char_array_4[0] << 2) + ((char_array_4[1] & 0x30) >> 4);
				char_array_3[1] = ((char_array_4[1] & 0xf) << 4) + ((char_array_4[2] & 0x3c) >> 2);
				char_array_3[2] = ((char_array_4[2] & 0x3) << 6) + char_array_4[3];

				for (i = 0; (i < 3); i++)
					ret += char_array_3[i];
				i = 0;
			}
		}

		if (i) {
			for (j = i; j < 4; j++)
				char_array_4[j] = 0;

			for (j = 0; j < 4; j++)
				char_array_4[j] = static_cast<unsigned char>(base64_chars.find(char_array_4[j]));

			char_array_3[0] = (char_array_4[0] << 2) + ((char_array_4[1] & 0x30) >> 4);
			char_array_3[1] = ((char_array_4[1] & 0xf) << 4) + ((char_array_4[2] & 0x3c) >> 2);
			char_array_3[2] = ((char_array_4[2] & 0x3) << 6) + char_array_4[3];

			for (j = 0; (j < i - 1); j++) ret += char_array_3[j];
		}

		return ret;
	}
};

```

# 56. ostringstream

```
#include <string>
#include <list>
#include <vector>
#include <memory>
#include <sstream>
#include <algorithm>
#include <iomanip>
#include <utility>
#include <set>
#include <cstring>
#include <cstdio>

class IExportable {
public:
	virtual std::string ToJson() = 0;
	virtual std::string ToXml() = 0;
	virtual std::string ToCsv() = 0;
	virtual std::string ToText() = 0;
};

class OleSummary : public IExportable {
public:
	std::string FullName;
	unsigned long long Size;

	virtual std::string ToJson() {
		std::ostringstream str;
		str << "{ \"path\" : \"" << helper::JsonEscape(this->FullName) << "\", \"size\" : \"" << this->Size << "\"}";
		return str.str();
	}
	virtual std::string ToXml() {
		std::ostringstream str;
		str << "<item>";
		str << "<path>" << this->FullName << "</path>";
		str << "<size>" << this->Size << "</size>";
		str << "</item>";
		return str.str();
	}
	virtual std::string ToText() {
		std::ostringstream str;
		str << this->FullName << "\t" << this->Size;
		return str.str();
	}
	virtual std::string ToCsv() {
		std::ostringstream str;
		str << this->FullName << "," << this->Size;
		return str.str();
	}
};

class ExtensionInfo : public IExportable {
public:
	std::string Extension;
	unsigned short Version;
	std::string VersionName;
public:
	virtual std::string ToJson() {
		std::ostringstream str;
		str << "{ \"extension\" : \"" << this->Extension << "\", \"version\" : \"" << this->Version << "\", \"name\" : \"" << this->VersionName << "\"}";
		return str.str();
	}
	virtual std::string ToXml() {
		std::ostringstream str;
		str << "<item>";
		str << "<extension>" << this->Extension << "</extension>";
		str << "<version>" << this->Version << "</version>";
		str << "<name>" << this->VersionName << "</name>";
		str << "</item>";
		return str.str();
	}
	virtual std::string ToText() {
		std::ostringstream str;
		str << this->Extension << "\t" << this->Version << "\t" << this->VersionName;
		return str.str();
	}
	virtual std::string ToCsv() {
		std::ostringstream str;
		str << this->Extension << "," << this->Version << "," << this->VersionName;
		return str.str();
	}
};



```


# 57 . declare DLL IMPORT or Export

## FBXSDK

```
#if defined(FBXSDK_SHARED)
	#if defined(FBXSDK_COMPILER_MSC) || defined(FBXSDK_COMPILER_INTEL)
		#define FBXSDK_DLLIMPORT __declspec(dllimport)
		#define FBXSDK_DLLEXPORT __declspec(dllexport)
	#elif defined(FBXSDK_COMPILER_GNU) && (__GNUC__ >= 4)
		#define FBXSDK_DLLIMPORT __attribute__((visibility("default")))
		#define FBXSDK_DLLEXPORT __attribute__((visibility("default")))
	#else
		#define FBXSDK_DLLIMPORT
		#define FBXSDK_DLLEXPORT
	#endif
#else
	#define FBXSDK_DLLIMPORT
	#define FBXSDK_DLLEXPORT
#endif
```

## FILEGDB
```
#ifndef EXPORT_FILEGDB_API
# if defined __linux__ || defined __APPLE__
#  define EXT_FILEGDB_API
# else
#  define EXT_FILEGDB_API _declspec(dllimport)
# endif
#else
# if defined __linux__ || defined __APPLE__
#  define EXT_FILEGDB_API __attribute__((visibility("default")))
# else
#  define EXT_FILEGDB_API _declspec(dllexport)
# endif
#endif
```


## PODOFO

```
#if defined(_WIN32)
    #if defined(COMPILING_SHARED_PODOFO)
        #define PODOFO_API __declspec(dllexport)
        #define PODOFO_DOC_API __declspec(dllexport)
	#elif defined(USING_SHARED_PODOFO)
		#define PODOFO_API __declspec(dllimport)
        #define PODOFO_DOC_API __declspec(dllimport)
    #else
        #define PODOFO_API
        #define PODOFO_DOC_API
    #endif
    /* PODOFO_LOCAL doesn't mean anything on win32, it's to exclude
     * symbols from the export table with gcc4. */
    #define PODOFO_LOCAL
#else
    #if defined(PODOFO_HAVE_GCC_SYMBOL_VISIBILITY)
        /* Forces inclusion of a symbol in the symbol table, so
           software outside the current library can use it. */
        #define PODOFO_API __attribute__ ((visibility("default")))
        #define PODOFO_DOC_API __attribute__ ((visibility("default")))
        /* Within a section exported with PODOFO_API, forces a symbol to be
           private to the library / app. Good for private members. */
        #define PODOFO_LOCAL __attribute__ ((visibility("hidden")))
        /* Forces even stricter hiding of methods/functions. The function must
         * absolutely never be called from outside the module even via a function
         * pointer.*/
        #define PODOFO_INTERNAL __attribute__ ((visibility("internal")))
    #else
        #define PODOFO_API
        #define PODOFO_DOC_API
        #define PODOFO_LOCAL
        #define PODOFO_INTERNAL
    #endif
#endif
```

## QT

```
#elif defined(__ARMCC__) || defined(__CC_ARM)
#  define Q_CC_RVCT
/* work-around for missing compiler intrinsics */
#  define __is_empty(X) false
#  define __is_pod(X) false
#  define Q_DECL_DEPRECATED __attribute__ ((__deprecated__))
#  ifdef Q_OS_LINUX
#    define Q_DECL_EXPORT     __attribute__((visibility("default")))
#    define Q_DECL_IMPORT     __attribute__((visibility("default")))
#    define Q_DECL_HIDDEN     __attribute__((visibility("hidden")))
#  else
#    define Q_DECL_EXPORT     __declspec(dllexport)
#    define Q_DECL_IMPORT     __declspec(dllimport)
#  endif
```


# 58. character string Converter

```
//// charsConverter.h //////
class CharsConverter
{
public:
	wchar_t *a2w(const char *pszSrc);
	char *   w2a(const wchar_t *wcharStr);
public:
	CharsConverter();
	~CharsConverter();
private:
	long     m_hchar;
	long	 m_hwchar;
};

#define USE_CHAR_CONVERTER	CharsConverter  g_char_string_converter;
#define A2W(a)	    g_char_string_converter.a2w(a)
#define W2A(w)    	g_char_string_converter.w2a(w)


//// charsConverter.cpp //////

#include <sys/types.h>
#include <unistd.h>

#include <stdlib.h>
#include <stdio.h>

#include <time.h>
#include <unistd.h>
#include <dlfcn.h>
#include <errno.h>
#include <fcntl.h>
#include <dirent.h>
#include <sys/stat.h>
#include <sys/ipc.h>
#include <sys/sem.h>
#include <pthread.h>

#include <wchar.h>
#include <locale.h>

#include <string>
#include <list>

typedef list<char *> charList;
typedef list<wchar_t *> wcharList;

CharsConverter::CharsConverter()
{
	m_hchar = 0;
	m_hwchar = 0;

	charList *pcharList = new charList();
	if (pcharList != NULL)
	{
		m_hchar = (long)pcharList;
	}
	wcharList *pwcharList = new wcharList();
	if (pwcharList != NULL)
	{
		m_hwchar = (long)pwcharList;
	}
}
CharsConverter::~CharsConverter()
{
	charList::iterator strchar;
	wcharList::iterator strwchar;

	charList *pcharList = (charList *)m_hchar;
	if(pcharList!=NULL)
	{
		for ( strchar = pcharList->begin( ); strchar != pcharList->end( ); strchar++ )
		{
			if (*strchar != NULL) delete[](char *)(*strchar);
			*strchar = NULL;
		}

		delete pcharList;
		pcharList=NULL;
		m_hchar = 0;
	}
	wcharList *pwcharList = (wcharList *)m_hwchar;
	if(pwcharList!=NULL)
	{
		for ( strwchar = pwcharList->begin( ); strwchar != pwcharList->end( ); strwchar++ )
		{
			if (*strwchar != NULL) delete[](char *)(*strwchar);
			*strwchar = NULL;
		}

		delete pwcharList;
		pwcharList=NULL;
		m_hwchar = 0;
	}
}
wchar_t *CharsConverter::a2w(const char *pszSrc)
{
	if(pszSrc==NULL)
		return NULL;

	wchar_t* pwcs = NULL; 
	int size = 0; 
	setlocale(LC_ALL, "zh_CN.GB2312"); 
	size = mbstowcs(NULL,pszSrc,0); 
	if (size <= 0) return NULL;

	pwcs = new wchar_t[size+1]; 
	if (pwcs == NULL) return NULL;
	size = mbstowcs(pwcs, pszSrc, size+1); 
	pwcs[size] = 0; 

	wcharList *pwcharList = (wcharList *)m_hwchar;
	if (pwcharList)
	{
		pwcharList->push_back( pwcs );
	}

	return pwcs; 
}
char *CharsConverter::w2a(const wchar_t *wcharStr)
{
	if(wcharStr==NULL)
		return NULL;

	char* str = NULL; 
	int   size = 0; 
	setlocale(LC_ALL, "zh_CN.UTF8"); 
	size = wcstombs( NULL, wcharStr, 0); 
	str = new char[size + 1]; 
	if (str == NULL) return NULL;

	wcstombs( str, wcharStr, size); 
	str[size] = '\0';

	charList *pcharList = (charList *)m_hchar;
	if (pcharList)
	{
		pcharList->push_back( str );
	}

	return  str; 
}

```

# 59.   __uuid(T)  

```
#include <iostream>
#include <memory.h>
#include <assert.h>

using namespace std;

#define nybble_from_hex(c)      ((c>='0'&&c<='9')?(c-'0'):((c>='a'&&c<='f')?(c-'a' + 10):((c>='A'&&c<='F')?(c-'A' + 10):0)))
#define byte_from_hex(c1, c2)   ((nybble_from_hex(c1)<<4)|nybble_from_hex(c2))

typedef struct _GUID {
	unsigned long  Data1;
	unsigned short Data2;
	unsigned short Data3;
	unsigned char  Data4[8];
} GUID;

static GUID GUID_NULL = { 0, 0, 0,{ 0, 0, 0, 0, 0, 0, 0 ,0 } };

#ifndef _REFGUID_DEFINED
#define _REFGUID_DEFINED
#ifdef __cplusplus
#define REFGUID const GUID &
#else
#define REFGUID const GUID * __MIDL_CONST
#endif
#endif

__inline int IsEqualGUID(REFGUID rguid1, REFGUID rguid2)
{
	return !memcmp(&rguid1, &rguid2, sizeof(GUID));
}

#ifdef __cplusplus
__inline bool operator==(REFGUID guidOne, REFGUID guidOther)
{
	return !!IsEqualGUID(guidOne, guidOther);
}

__inline bool operator!=(REFGUID guidOne, REFGUID guidOther)
{
	return !(guidOne == guidOther);
}
#endif

struct Initializable : public GUID
{
	explicit
		Initializable(char const (&spec)[37])
		: GUID()
	{
		for (int i = 0; i < 8; ++i)
		{
			Data1 = (Data1 << 4) | nybble_from_hex(spec[i]);
		}
		assert(spec[8] == '-');
		for (int i = 9; i < 13; ++i)
		{
			Data2 = (Data2 << 4) | nybble_from_hex(spec[i]);
		}
		assert(spec[13] == '-');
		for (int i = 14; i < 18; ++i)
		{
			Data3 = (Data3 << 4) | nybble_from_hex(spec[i]);
		}
		assert(spec[18] == '-');
		for (int i = 19; i < 23; i += 2)
		{
			Data4[(i - 19) / 2] = (nybble_from_hex(spec[i]) << 4) | nybble_from_hex(spec[i + 1]);
		}
		assert(spec[23] == '-');
		for (int i = 24; i < 36; i += 2)
		{
			Data4[2 + (i - 24) / 2] = (nybble_from_hex(spec[i]) << 4) | nybble_from_hex(spec[i + 1]);
		}
	}
};

template<class T>
inline
auto __my_uuidof()
-> GUID const &
{
	return GUID_NULL;
}

#define CPPX_GNUC_UUID_FOR( name, spec )            \
template<>                                          \
inline                                              \
auto __my_uuidof<name>()                            \
    -> GUID const&                                  \
{                                                   \
    using ::operator"" _uuid;                       \
    static GUID the_uuid = spec ## _uuid;           \
                                                    \
    return the_uuid;                                \
}                                                   \
                                                    \
template<>                                          \
inline                                              \
auto __my_uuidof<name*>()                           \
    -> GUID const&                                  \
{ return __my_uuidof<name>(); }                     \
                                                    \
static_assert( true, "" )

auto operator"" _uuid(char const* const s, size_t const size)
-> GUID
{
	return Initializable(reinterpret_cast<char const (&)[37]>(*s));
}

#define CPPX_UUID_FOR    CPPX_GNUC_UUID_FOR

#define __uuid(T)       __my_uuidof<T>()

struct Foo {};
CPPX_UUID_FOR(Foo, "dbe41a75-d5da-402a-aff7-cd347877ec00");

Foo foo;

template <class T>
void QueryInterface(T** p)
{
	if (p == NULL)
		return;

	GUID guid = "dbe41a75-d5da-402a-aff7-cd347877ec00"_uuid;
	if (__uuid(T) == guid)
	{
		*p = &foo;
	}
	else
	{
		*p = NULL;
	}

	return;
}

int testGUID()
{
	Foo* p = NULL;
	QueryInterface(&p);

	cout << "p: " << p << ", &foo: " << &foo << endl;

	return 0;
}

```

# 60. loadlibrary and GetSystemInfo

```

#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <time.h>
#include <unistd.h>
#include <dlfcn.h>
#include <errno.h>
#include <fcntl.h>
#include <dirent.h>
#include <sys/stat.h>
#include <sys/ipc.h>
#include <sys/sem.h>
#include <pthread.h>
#include <wchar.h>
#include <locale.h>
#include <string>
#include <list>
#include <err.h>
#include <pwd.h>

#if defined(_IOS_)
#include <sys/sysctl.h>
#else
#include <sys/sysinfo.h>
#include <malloc.h>
#endif
#include <unistd.h>
#include <iphlpapi.h>
#include <sys/statvfs.h>

#ifndef STDAPI
#define STDAPI __stdcall
#endif

typedef INT_PTR (FAR STDAPI *FARPROC)();
typedef PVOID HLIBMODULE
BOOL STDAPI FreeLibrary (  HLIBMODULE hLibModule )
{
	return(FALSE);
}

char * str2lower(char *dst)
{
	if(dst==NULL)
		return (NULL);		

	int i=0;
	while(dst[i]!='\0')		
	{
		if(dst[i]>='A' && dst[i]<='Z')
			dst[i] +=32;
		i++;
	}

	return (dst);
}

HLIBMODULE STDAPI LoadLibraryA(LPCSTR pszFullFileName)
{
 	char szPath[MAX_PATH]		= {0};
	std::string strFileName			= pszFullFileName;
	std::string strDllName;
	long	nPos				= strFileName.find_last_of("/");
	strDllName = strFileName.substr(nPos+1,strFileName.length()-nPos);
 	strcpy(szPath,strDllName.c_str());
 	std::string path = str2lower(szPath);
	std::string name;
	if (strcasecmp(path.substr(path.length()-4,4).c_str(),".dll")==0)
	{
		long pos=path.rfind('/');
		if(pos>=0)
		{
			name=path.substr(0,pos+1);
			name+="lib";
			name+=path.substr(pos+1,path.length()-3-(pos+1));
			name+="so";
		}
		else
		{
			name="lib";
			name+=path.substr(0,path.length()-3);
			name+="so";
		}
	}
	else
		name=path;

	return((HLIBMODULE)dlopen(name.c_str(),RTLD_LAZY));
}
FARPROC STDAPI GetProcAddress (  HLIBMODULE hModule,  LPCSTR lpProcName )
{
	return((FARPROC)dlsym(hModule,lpProcName));
}

#if defined(_IOS_)
VOID STDAPI GetSystemInfo(  LPSYSTEM_INFO pSysInfo )
{
	if(!pSysInfo) return;
    size_t length = 0;
    static const int names[] ={CTL_KERN, KERN_PROC, KERN_PROC_ALL, 0};
    sysctl((int *)names, (sizeof(names)/sizeof(names[0]))-1, NULL, &length, NULL, 0);
    pSysInfo->dwPageSize = sysconf(_SC_PAGESIZE);
    pSysInfo->dwAllocationGranularity = 64000; 
    pSysInfo->dwNumberOfProcessors = (length/sizeof(kinfo_proc));
}
#else
VOID STDAPI GetSystemInfo(  LPSYSTEM_INFO pSysInfo )
{
	if(!pSysInfo) return;
    struct sysinfo   sys1;
    sysinfo(&sys1);
    pSysInfo->dwPageSize = sysconf(_SC_PAGESIZE);
    pSysInfo->dwAllocationGranularity = 64000; 
    pSysInfo->dwNumberOfProcessors = sys1.procs;
}
#endif


```



# 61. reference.h

```

#ifndef __REFERENCE_HEADER_DEFINED
#define __REFERENCE_HEADER_DEFINED

#include <stdio.h>
#include <stdarg.h>
#include <stdlib.h>
#include <inttypes.h>
#include <stddef.h>

typedef struct IStream IStream;
typedef struct IStorage IStorage;

typedef struct _XGUID
{
	uint32_t	Data1;
	uint16_t	Data2;
	uint16_t	Data3;
	uint8_t	    Data4[8];
} XGUID, GUID, CLSID, IID;

#if !defined(WCHAR) && !defined(__BORLANDC__)
typedef uint16_t	WCHAR;
typedef WCHAR	*   LPWSTR;
#endif
typedef uint16_t	WORD;
typedef uint32_t	DWORD;


#ifndef FARSTRUCT
#define FARSTRUCT
#endif

#if !defined(WIN32)
typedef struct FARSTRUCT tagFILETIME
{
	DWORD dwLowDateTime;
	DWORD dwHighDateTime;
} FILETIME;
#endif

#ifndef TRUE
#define TRUE 1
#endif

#ifndef FALSE
#define FALSE 0
#endif

#define FARSTRUCT
#define interface struct
#define DECLARE_INTERFACE(iface) interface iface
#define DECLARE_INTERFACE_(iface, baseiface) interface iface: public baseiface

typedef int32_t SCODE;
typedef int32_t HRESULT;

#define NOERROR 0

#ifdef __cplusplus
    #define EXTERN_C    extern "C"
#else
    #define EXTERN_C    extern
#endif


typedef int           BOOL, *LPBOOL;
typedef BOOL          BOOLEAN;
typedef unsigned char BYTE;
typedef char          CHAR, *PCHAR;
typedef unsigned char UCHAR;
typedef uint32_t	  UINT;
typedef int32_t	      INT;
typedef int32_t	      LONG;
typedef int16_t	      SHORT;
typedef uint16_t	  USHORT;
typedef DWORD         ULONG;
typedef void          VOID;
typedef LONG          NTSTATUS, *PNTSTATUS;

typedef void *        LPVOID;
typedef char *        LPSTR;
typedef const char *  LPCSTR;


#  ifndef __int64
#    define __int64     long long
#  endif

typedef signed __int64  INT64, *PINT64;
typedef unsigned char   byte;
typedef DWORD           COLORREF, *LPCOLORREF;
typedef long            *LPLONG;
typedef void*           PVOID;
typedef PVOID           HFILE;
typedef PVOID           HBITMAP;
typedef PVOID           HDC;
typedef PVOID           HWND;
typedef PVOID           HANDLE;
typedef PVOID           HINSTANCE;
typedef PVOID           HMODULE;
typedef DWORD           *LPDWORD;
typedef GUID            _GUID;
typedef unsigned long   ULONG_PTR, *PULONG_PTR;
typedef ULONG_PTR       SIZE_T, *PSIZE_T;
typedef const char     *LPCTSTR;
typedef char            TCHAR, _TCHAR;
typedef char           *LPTSTR;


// NOTE: 
// for other compilers some form of 64 bit integer support 
// has to be provided
#ifdef _MSC_VER
typedef __int64                LONGLONG;
typedef unsigned __int64       ULONGLONG;
#else
// should work with most Unix compilers
// FIXME: portability
typedef long long int          LONGLONG;
typedef unsigned long long int ULONGLONG;
#endif

typedef union _LARGE_INTEGER {
	struct {
		DWORD LowPart;
		LONG HighPart;
	} u;
	LONGLONG QuadPart;
} LARGE_INTEGER, *PLARGE_INTEGER;

#define LISet32(li, v)     ((li).u.HighPart = ((LONG)(v)) < 0 ? -1 : 0, (li).u.LowPart = (v))
#define LISetLow(li, v)    ((li).u.LowPart = (v))
#define LISetHigh(li, v)   ((li).u.HighPart = (v))
#define LIGetLow(li)       ((li).u.LowPart)
#define LIGetHigh(li)      ((li).u.HighPart)



#endif ///__REFERENCE_HEADER_DEFINED

```



# 62. olechar.h

```

#ifndef __TCHAR_DEFINED
#define __TCHAR_DEFINED
#include "reference.hxx"

#ifdef OLE_UNICODE

typedef WCHAR TCHAR__;
#define OLESTR(str) L##str

#else  /* OLE_UNICODE */

typedef char TCHAR__;
#define OLESTR(str) str

#endif /* OLE_UNICODE */

typedef TCHAR__ * LPTSTR_;
typedef TCHAR__ OLECHAR, *LPOLECHAR, *LPOLESTR;

/* Define some macros to handle declaration of strings with literals */
/* Since we stray from the default 4 byte wchar_t in UNIX, we have to
   to something different than the usual L"ssd" literals */

/* #define DECLARE_OLESTR(ocsName, len, contents) \ */
/*         LPOLESTR ocsName[len]=OLESTR(contents) */

/* #else */

#ifdef OLE_UNICODE
#define DECLARE_OLESTR(ocsName, pchContents)                       \
        OLECHAR ocsName[sizeof(pchContents)+1];                    \
        _tbstowcs(ocsName, pchContents, sizeof(pchContents)+1)     

#define INIT_OLESTR(ocsName, pchContents) \
        _tbstowcs(ocsName, pchContents, sizeof(pchContents)+1)     

#define DECLARE_CONST_OLESTR(cocsName, pchContents)                     \
        OLECHAR temp##cocsName[sizeof(pchContents)+1];                  \
        _tbstowcs(temp##cocsName, pchContents, sizeof(pchContents)+1);  \
        const LPOLESTR cocsName = temp##cocsName
        
#else  /* non  OLE_UNICODE */

#define DECLARE_OLESTR(ocsName, pchContents)             \
        OLECHAR ocsName[]=pchContents

#define INIT_OLESTR(ocsName, pchContents) \
        strcpy(ocsName, pchContents);

#define DECLARE_CONST_OLESTR(ocsName, pchContents)       \
        const LPOLESTR ocsName=pchContents 

#endif /* OLE_UNICODE */

#define DECLARE_WIDESTR(wcsName, pchContents)                      \
        WCHAR wcsName[sizeof(pchContents)+1];                      \
        _tbstowcs(wcsName, pchContents, sizeof(pchContents)+1)



#ifndef OLE_UNICODE                /*---- non unicode ------  */

#define _tcscpy   strcpy
#define _tcscmp   strcmp
#define _tcslen   strlen
#define _tcsnicmp _strnicmp
#define _tcscat   strcat
#define _itot     _itoa
#define _T(str)   str

#ifdef _WIN32

/* Io functions */
#define _tfopen    fopen
#define _tunlink   _unlink
#define _tfullpath _fullpath
#define _tstat     _stat

#else /* _WIN32 */

#define _tfopen   fopen
#define _tunlink  unlink        /* T-types mapping */
#define _unlink   unlink        /* non-win32 mapping */
#define _stat stat
#define _tstat stat
#define _strnicmp(s1,s2,n) strncasecmp(s1,s2,n)
 
/* note that we assume there is enough space in this case */
#define _tfullpath(longname, shortname, len)    realpath(shortname, longname) 
#define _fullpath(longname, shortname, len)    realpath(shortname, longname) 

#endif /* _MSC_VER */

/* copying wchar/char and TCHAR__ */
#ifdef _MSC_VER
#define WTOT(T, W, count) wcstombs(T, W, count) 
#define TTOW(T, W, count) mbstowcs(W, T, count)
#else /* _MSC_VER */
#define WTOT(T, W, count) wcstosbs(T, W, count)
#define TTOW(T, W, count) sbstowcs(W, T, count)
#endif /* _MSC_VER */

#define STOT(S, T, count) strcpy(T, S)
#define TTOS(T, S, count) strcpy(S, T)

#else                          /* OLE_UNICODE   ---- unicode  ------ */

/* NOTE: unicode APIs on non win32 systems are not tested or implemented */

#define _tcscpy   wcscpy
#define _tcscmp   wcscmp
#define _tcslen   wcslen
#define _tcscat   wcscat
#define _tcsnicmp wcsnicmp
#define _itot     _itow
#define _T(str)   L##str

/* Io functions */
#define _tfopen    _wfopen
#define _tunlink   _wunlink
#define _tfullpath _wfullpath
#define _tstat     _wstat

#ifdef _UNIX                    /* map win32 I/O API's to other O.S. */
#define _unlink unlink
#define _fullpath(longname, shortname, len)    realpath(shortname, longname) 
#define _stat stat
#define _strnicmp(s1,s2,n) strncasecmp(s1,s2,n)
#endif

/* converting between wchar and TCHAR__ */
#define WTOT(T, W, count) wcsncpy(T, W, count) 
#define TTOW(T, W, count) wcsncpy(W, T, count)

/* converting between a char and TCHAR__ */
#define WTOT(T, W, count) wcsncpy(T, W, count) 
#define TTOW(T, W, count) wcsncpy(W, T, count)

#define STOT(S, T, count) _tbstowcs(T, S, count)
#define TTOS(T, S, count) _wcstotbs(S, T, count)

#endif /* #ifndef OLE_UNICODE, #else ... */



#ifndef _WIN32                /* others */
#define _tbstowcs sbstowcs
#define _wcstotbs wcstosbs 
#else /* _WIN32 */
#define _tbstowcs mbstowcs
#define _wcstotbs wcstombs 
#endif /* _WIN32 */

#ifndef _MSC_VER
#include <assert.h>   
inline void  _itoa(int v, char* string, int radix)
{
  if (radix!=10) assert(FALSE);  /* only handle base 10 */
  sprintf(string, "%d", v);
}
#endif /* _MSC_VER */


#define ocslen      _tcslen
#define ocscpy      _tcscpy
#define ocscmp      _tcscmp
#define ocscat      _tcscat
#define ocschr      _tcschr
#define soprintf    _tprintf
#define oprintf     _tprintf
#define ocsnicmp    _tcsnicmp


#endif  /* #ifndef __TCHAR_DEFINED */

```

# 63. time.hxx

```

#include <time.h>
#include <limits.h>
#include <assert.h>
#include "reference.hxx"


// Time type
typedef FILETIME TIME_T;
#define STDAPI_(type)  type

// Number of seconds difference betwen FILETIME (since 1601 00:00:00) 
// and time_t (since 1970 00:00:00)
//
// This should be a constant difference between the 2 time formats
//
#ifdef __GNUC__ // cater to differences of compiler syntax
const LONGLONG ci64DiffFTtoTT=11644473600LL; 
#else          //  __GNUC__
const LONGLONG ci64DiffFTtoTT=11644473600;
#endif         //  __GNUC__

STDAPI_(void) FileTimeToTimeT(const FILETIME *pft, time_t *ptt)
{
    ULONGLONG llFT = pft->dwHighDateTime;
    llFT = (llFT << 32) | (pft->dwLowDateTime);
    // convert to seconds 
    // (note that all fractions of seconds will be lost)
    llFT = llFT/10000000;       
    llFT -= ci64DiffFTtoTT;         // convert to time_t 
    assert(llFT <= ULONG_MAX);
    *ptt = (time_t) llFT;
}

STDAPI_(void) TimeTToFileTime(const time_t *ptt, FILETIME *pft)
{
    ULONGLONG llFT = *ptt;
    llFT += ci64DiffFTtoTT;         // convert to file time
    // convert to nano-seconds
    for (int i=0; i<7; i++)         // mulitply by 10 7 times
    {        
        llFT = llFT << 1;           // llFT = 2x
        llFT += (llFT << 2);        // llFT = 4*2x + 2x = 10x
    }
    pft->dwLowDateTime  = (DWORD) (llFT & 0xffffffff);
    pft->dwHighDateTime = (DWORD) (llFT >> 32);
}

inline SCODE CoFileTimeNow(TIME_T *pft)
{
    time_t tt;
    time(&tt);
    TimeTToFileTime(&tt, pft);
    return S_OK;
}


```


```
#include <stdlib.h>

#include <sys/stat.h>
#include "time.hxx"
#include "h/tchar.h"

#ifdef _WIN32
#include <io.h> // to get definition of wunlink
#else
#include <unistd.h>
// get the correct stat structure
#endif

#include <stdlib.h>

typedef struct FARSTRUCT tagSTATSTG
{
    TCHAR__ FAR* pwcsName;
    DWORD type;
    ULARGE_INTEGER cbSize;
    FILETIME mtime;
    FILETIME ctime;
    FILETIME atime;
    DWORD grfMode;
    DWORD grfLocksSupported;
    CLSID clsid;
    DWORD grfStateBits;
    DWORD reserved;
} STATSTG;

STATSTG*  pstatstg = new STATSTG();

char *_pszName="/home/memory.io";    
// we always use ANSI char when dealing with
struct _stat buf;
int result = _stat(_pszName, &buf);
if (!result)  // fill in zeros
{
	pstatstg->atime.dwLowDateTime = 0;
	pstatstg->mtime.dwLowDateTime = 0;
	pstatstg->ctime.dwLowDateTime = 0;
}
else
{
	TimeTToFileTime(&buf.st_atime, &pstatstg->atime);
	TimeTToFileTime(&buf.st_mtime, &pstatstg->mtime);
	TimeTToFileTime(&buf.st_ctime, &pstatstg->ctime);
}
delete pstatstg;


```



# 64. compress and decompress

```
#include "zlib.h"

// https://github.com/mapbox/mapnik-vector-tile/blob/master/src/vector_tile_compression.hpp
bool is_compressed(std::string const &data) {
	return data.size() > 2 && (((uint8_t) data[0] == 0x78 && (uint8_t) data[1] == 0x9C) || ((uint8_t) data[0] == 0x1F && (uint8_t) data[1] == 0x8B));
}

// https://github.com/mapbox/mapnik-vector-tile/blob/master/src/vector_tile_compression.hpp
int decompress(std::string const &input, std::string &output) {
	z_stream inflate_s;
	inflate_s.zalloc = Z_NULL;
	inflate_s.zfree = Z_NULL;
	inflate_s.opaque = Z_NULL;
	inflate_s.avail_in = 0;
	inflate_s.next_in = Z_NULL;
	if (inflateInit2(&inflate_s, 32 + 15) != Z_OK) {
		fprintf(stderr, "error: %s\n", inflate_s.msg);
	}
	inflate_s.next_in = (Bytef *) input.data();
	inflate_s.avail_in = input.size();
	size_t length = 0;
	do {
		output.resize(length + 2 * input.size());
		inflate_s.avail_out = 2 * input.size();
		inflate_s.next_out = (Bytef *) (output.data() + length);
		int ret = inflate(&inflate_s, Z_FINISH);
		if (ret != Z_STREAM_END && ret != Z_OK && ret != Z_BUF_ERROR) {
			fprintf(stderr, "error: %s\n", inflate_s.msg);
			return 0;
		}

		length += (2 * input.size() - inflate_s.avail_out);
	} while (inflate_s.avail_out == 0);
	inflateEnd(&inflate_s);
	output.resize(length);
	return 1;
}

// https://github.com/mapbox/mapnik-vector-tile/blob/master/src/vector_tile_compression.hpp
int compress(std::string const &input, std::string &output) {
	z_stream deflate_s;
	deflate_s.zalloc = Z_NULL;
	deflate_s.zfree = Z_NULL;
	deflate_s.opaque = Z_NULL;
	deflate_s.avail_in = 0;
	deflate_s.next_in = Z_NULL;
	deflateInit2(&deflate_s, Z_BEST_COMPRESSION, Z_DEFLATED, 31, 8, Z_DEFAULT_STRATEGY);
	deflate_s.next_in = (Bytef *) input.data();
	deflate_s.avail_in = input.size();
	size_t length = 0;
	do {
		size_t increase = input.size() / 2 + 1024;
		output.resize(length + increase);
		deflate_s.avail_out = increase;
		deflate_s.next_out = (Bytef *) (output.data() + length);
		int ret = deflate(&deflate_s, Z_FINISH);
		if (ret != Z_STREAM_END && ret != Z_OK && ret != Z_BUF_ERROR) {
			return -1;
		}
		length += (increase - deflate_s.avail_out);
	} while (deflate_s.avail_out == 0);
	deflateEnd(&deflate_s);
	output.resize(length);
	return 0;
}


std::string decode(std::string &message, bool &was_compressed) {
	std::string src;

	if (is_compressed(message)) {
		std::string uncompressed;
		decompress(message, uncompressed);
		src = uncompressed;
		was_compressed = true;
	} else {
		src = message;
		was_compressed = false;
	}
	return src;
}

```


# 65. ifstream and ofstream

```

include  <stdio.h>
#include <sys/types.h>
#include <iostream>
#include <fstream>
#include <sstream>
#include <string>
#include <sys/stat.h>

#if defined(_WIN32)
#include <afx.h>
#include <atlconv.h>
#define mkdirectory(path) CreateDirectory(path, NULL)
#define SLASH "\\"
#else
#define mkdirectory(path) mkdir(path, S_IRWXU | S_IRWXG | S_IRWXO)
#define SLASH "/"
#endif

std::string Read(std::string strPath) {
	std::ifstream strFile(strPath, std::ios::in | std::ios::binary);
	std::ostringstream contents;
	contents << strFile.rdbuf();
	strFile.close();

	return (contents.str());
}

void Write(const char *outdir, int z, int tx, int ty, std::string const &buff) {
	mkdirectory(outdir);
	std::string curdir(outdir);
	std::string slash(SLASH);
	std::string newdir = curdir + slash + std::to_string(z);
	mkdirectory(newdir.c_str());
	newdir = newdir + SLASH + std::to_string(tx);
	mkdirectory(newdir.c_str());
	newdir = newdir + SLASH + std::to_string(ty) + ".buf";

	std::ofstream strFile(newdir, std::ios::out | std::ios::binary);
	strFile.write(buff.data(), buff.size());
	strFile.close();
}


```



# 66. disperse ellipse arc

```

//==============================DisperseEllipseArc()==============================
/// @brief 椭圆弧段离散化。
///
/// 椭圆弧段离散化。
///
/// @param [out] ddArcBuff  离散化后的点序列。
/// @param [out] lBuflen    离散化后点的个数。
/// @param [in]  dStep      离散化的步长(弧度单位)。
/// @param [in]  center     椭圆中心点。
/// @param [in]  dAxis_a    椭圆a半径。
/// @param [in]  dAxis_b    椭圆b半径。
/// @param [in]  dAngle     椭圆倾斜角度（弧度，以椭圆中心点为中心，逆时针转）
/// @param [in]  beginAngle 起始弧度。
/// @param [in]  endAngle   终止弧度。
///
/// @return <=0 失败，>0成功。
//================================================================================
// Revision history : 1
// pay attention to the special case: the end angle is less than the begin angle;
// fix the case by dividing into 2 blocks,modified by cheldon ,2018.10.11
// Revision history : 2
// make sure the beginAngle and EndAngle is In rad unit, not in degree unit,
// maybe need some flag to make difference between rad and degree,by cheldon,2018.11.1

long DisperseEllipseArc(DOT **ddArcBuff, long& lBuflen, double dStep, DOT &center, double dAxis_a, double dAxis_b, double dAngle, double beginAngle, double endAngle)
{
	long rtn = 0;
	if (dStep <= ZERO_FLOAT)	return 0;

	double a0 = fabs(dAxis_a);
	double b0 = fabs(dAxis_b);
	if (a0 < 1 && b0 < 1) return rtn;
	
	const double dloop = 2 * PI;
	while (beginAngle - dloop > ZERO_FLOAT) {
		beginAngle -= dloop;
	}
	while (endAngle - dloop > ZERO_FLOAT) {
		endAngle -= dloop;
	}

	double	dAngBegin = beginAngle, dAngEnd = endAngle;
	int nNum = static_cast<int>((2 * PI) / dStep);
	int nBegin = static_cast<int>(dAngBegin / dStep);
	int nEnd = static_cast<int>(dAngEnd / dStep);

	if (nBegin < nEnd)
	{
		lBuflen = nEnd - nBegin + 1;
	}
	else
	{
		lBuflen = nNum - (nBegin - nEnd) + 1;
	}

	*ddArcBuff = new DOT[lBuflen + 1];//

	double	cosa = cos(dAngle), sina = sin(dAngle);
	double	ang = 0; double x1 = 0, y1 = 0;
	double   x = a0*cos(ang);
	double   y = b0*sin(ang);
	int   nCount = 0;
	for (int nIndex = 0; nIndex <= nNum; nIndex++)
	{
		x = a0*cos(ang);
		y = b0*sin(ang);
		x1 = cosa*x - sina*y + center.x;
		y1 = sina*x + cosa*y + center.y;

		//if begin < end , num = end -begin; else divide into 2 blocks:[begin,2*PI] [0,end];
		if (nBegin <= nEnd && (nIndex >= nBegin && nIndex <= nEnd))
		{
			(*ddArcBuff)[nCount].x = x1;
			(*ddArcBuff)[nCount].y = y1;
			nCount++;
		}
		else if (nBegin > nEnd && (nIndex >= 0 && nIndex <= nEnd))
		{
			int  nOffset = nIndex + nNum - nBegin;
			(*ddArcBuff)[nOffset].x = x1;
			(*ddArcBuff)[nOffset].y = y1;
		}
		else if (nBegin > nEnd && (nIndex >= nBegin && nIndex <= nNum))
		{
			(*ddArcBuff)[nCount].x = x1;
			(*ddArcBuff)[nCount].y = y1;
			nCount++;
		}

		ang += dStep;
	}

	rtn = lBuflen;
	return rtn;
}
```


# 67. MoveFirst and MoveNext for set

```
#define  BEFORE_OF_SET  -1
#define  END_OF_SET     -10000

class Model;

struct tagModelSet
{
	std::vector<Model*> set;
	long        		 cursor;
};

class  ModelSet
{
public:
	ModelSet();
	virtual ~ModelSet();

	Model*	New();
	long    MoveFirst();
	long	MoveNext();
	Model*	Get();
	long	RemoveAll();
	long    GetSize();
private:
	tagModelSet	*m_pObj;
};



ModelSet::ModelSet()
{
	m_pObj=new tagModelSet;
	m_pObj->cursor=-1;
}

ModelSet::~ModelSet()
{
	RemoveAll();	
	delete m_pObj;
}

Model*	ModelSet::New()
{
	Model *pt=new Model;
	m_pObj->set.push_back(pt);
	return pt;
}

long		ModelSet::MoveFirst()
{
	if(m_pObj->set.size()==0) return END_OF_SET;
	m_pObj->cursor=0;
	return 1;
}

long		ModelSet::MoveNext()
{
	if(m_pObj->set.size()<=m_pObj->cursor+1) return END_OF_SET;
	m_pObj->cursor++;
	return 1;
}

Model*	ModelSet::Get()
{
	return m_pObj->set[m_pObj->cursor];
}

long		ModelSet::RemoveAll()
{
	for(long i=0;i<m_pObj->set.size();i++)
		delete m_pObj->set[i];
	m_pObj->set.clear();
	m_pObj->cursor=-1;
	return 1;
}

long ModelSet::GetSize()
{
	if (m_pObj == NULL)
		return 0;

	return m_pObj->set.size();
}

////////////////////////////////////
/// How to use model set
///////////////////////////////////
void TestModelSet()
{
	ModelSet models;
	Model * pNewModel = models.New();
	////////////////////////////////////
	//// update model
	///////////////////////////////////
	if (models.MoveFirst() == END_OF_SET)
	  return;
	do 
	{
		Model*	pModel = models.Get();
		if (!pModel)
			continue;
	    //// do something  

	} while (models.MoveNext() != END_OF_SET);

}


```


# 68. create the new object smartly

```
typedef std::basic_stringstream<char,std::char_traits<char>,std::allocator<char> > StringStream;
typedef std::map<std::string, Model* > ModelMap;

static  long nGenNextName = 0;
Model* CreateModel(const std::string& modelName, ModelType type)
{
	if (modelName.size() == 0)
	{
		StringStream str;
		str << "name_" << nGenNextName++;
		modelName = str.str();
	}

	///check whether find the target name
	/// if find, return the target
	/// if not, create new one and push it into cache;

	ModelMap::iterator iter = mModels.find(modelName);
	if (iter != mModels.end())
	{
		if (iter->second->mType == type)
		{
			return iter->second;
		}
		else
		{
			return NULL;
		}
	}

	Model* pModel = new Model(modelName, type);

	mModels[modelName] = pModel;
	return pModel;
	
}

```

# 69. mutex 

```
///////////////////////////////////////////

void *Thread_CreateMutex();
void  Thread_DestroyMutex( void *phMutex );
int   Thread_AcquireMutex( void *phMutex, double dfWaitInSeconds );
void  Thread_ReleaseMutex( void *phMutex );

#define Thread_MutexHolder(x)  MutexHolder mutex_holder(x,-1.0);

class MutexHolder
{
public:
	MutexHolder( void *phMutex, double dfWaitInSeconds = -1.0);
	~MutexHolder();

private:
	void       *m_phMutex;  ///Thread_CreateMutex
};



#ifndef MIN
#define MIN(a,b)            (((a) < (b)) ? (a) : (b))
#endif

///////////////////////////////////////////

#ifdef __linux__
#include <pthread.h>
#endif

void *Thread_CreateMutex()
{

#ifndef __linux__
	CRITICAL_SECTION *pcs = NULL;
	pcs = (CRITICAL_SECTION *)malloc(sizeof(*pcs));
	if (pcs)
	{
		InitializeCriticalSectionAndSpinCount(pcs, 4000);
		return (void*)pcs;
	}
	return NULL;

#else
	pthread_mutex_t mutex;
	pthread_mutex_init(&mutex, NULL);
	return (void*)&mutex;
	//pthred_mutexattr_init(mutex->attr);
	//pthred_mutexattr_settype(mutex->attr,PTHREAD_MUTEX_RECURSIVE);
	//pthred_mutex_init(mutex->mtx, mutex->attr);
#endif

}

void  Thread_DestroyMutex( void *phMutex )
{
#ifndef __linux__
	CRITICAL_SECTION *pcs = (CRITICAL_SECTION *)phMutex;
	DeleteCriticalSection(pcs);
	free(pcs);
#else
	pthread_mutex_t *mutex = (pthread_mutex_t *)phMutex;
	pthread_mutex_destroy(mutex);
#endif	
}

int   Thread_AcquireMutex( void *phMutex, double dfWaitInSeconds )
{
#ifndef __linux__
	CRITICAL_SECTION *pcs = (CRITICAL_SECTION *)phMutex;
	BOOL ret = FALSE;
	if (dfWaitInSeconds > 0.0) {
		while ((ret = TryEnterCriticalSection(pcs)) == 0 && dfWaitInSeconds > 0.0)
		{
			Sleep((DWORD)(MIN(dfWaitInSeconds, 0.125) * 1000.0));
			dfWaitInSeconds -= 0.125;
		}

		return ret;
		}
	else {
		EnterCriticalSection(pcs);
		return 1;
	}
	return 0;
#else
	pthread_mutex_t *mutex = (pthread_mutex_t *)phMutex;
	BOOL ret = FALSE;
	if (dfWaitInSeconds > 0.0) {
		while ((ret = pthread_mutex_trylock(mutex)) == 0 && dfWaitInSeconds > 0.0)
		{
			Sleep((DWORD)(MIN(dfWaitInSeconds, 0.125) * 1000.0));
			dfWaitInSeconds -= 0.125;
		}

		return ret;
	}
	else {
		pthread_mutex_lock(mutex);
		return 1;
	}
	return 0;
#endif
}

void  Thread_ReleaseMutex( void *phMutex )
{
#ifndef __linux__
	CRITICAL_SECTION *pcs = (CRITICAL_SECTION *)phMutex;
	LeaveCriticalSection(pcs);
#else
	pthread_mutex_t *mutex = (pthread_mutex_t *)phMutex;
	pthread_mutex_unlock(mutex);
#endif
}

MutexHolder::MutexHolder(void *phMutex, double dfWaitInSeconds)
{
	Thread_AcquireMutex(phMutex,dfWaitInSeconds);
	m_phMutex = phMutex;
}

MutexHolder::~MutexHolder()
{
	if (m_phMutex != NULL)
	{
		Thread_ReleaseMutex(m_phMutex);
		m_phMutex = NULL;
	}
}


```


# 70.  Hash string 


```
unsigned int       DecodeFixed32(const char* ptr);
unsigned int       Hash(const char* data, size_t n, unsigned int seed = 0);


#include <cstddef>
#include <cstdint>
unsigned int DecodeFixed32(const char* ptr)
{
	if (1)
	{
		// Load the raw bytes
		unsigned int result;
		memcpy(&result, ptr, sizeof(result));  // gcc optimizes this to a plain load
		return result;
	}
	else
	{
		return ((static_cast<unsigned int>(ptr[0]))
			| (static_cast<unsigned int>(ptr[1]) << 8)
			| (static_cast<unsigned int>(ptr[2]) << 16)
			| (static_cast<unsigned int>(ptr[3]) << 24));
	}
}

unsigned int Hash(const char* data, size_t n, unsigned int seed) /// leveldb-hash
{
	// Similar to murmur hash
	const unsigned int m = 0xc6a4a793;
	const unsigned int r = 24;
	const char* limit = data + n;
	unsigned int h = seed ^ (n * m);

	// Pick up four bytes at a time
	while (data + 4 <= limit) {
		unsigned int w = DecodeFixed32(data);
		data += 4;
		h += w;
		h *= m;
		h ^= (h >> 16);
	}

	// Pick up remaining bytes
	switch (limit - data) {
	case 3:
		h += static_cast<unsigned char>(data[2]) << 16;
		// fall through
	case 2:
		h += static_cast<unsigned char>(data[1]) << 8;
		// fall through
	case 1:
		h += static_cast<unsigned char>(data[0]);
		h *= m;
		h ^= (h >> r);
		break;
	}
	return h;
}


```

# 71.  compute the tail point base on the angle and length 


```
////////////////////ComputeTail//////////////////////////////////
/// [in]    x      |  the origin point x
/// [in]    y      |  the origin point y
/// [in]    angle  |  orientation angle
/// [in]    len    |  distance from the origin point
/// [out]   xTail  |  the tail point x
/// [out]   ytail  |  the tail point y  
///////////////////////////////////////////////////////////////

int  ComputeTail(double x, double y, double angle, double len, double& xTail, double& yTail)
{

	double dTanAng = tan(angle);
	double div = (sqrt(1 + dTanAng*dTanAng));

	if (fabs(angle - 0.5*PI) < 1E-5)
	{
		xTail = x;
		yTail = y + len;
	}
	else if (fabs(angle - 1.5*PI) < 1E-5)
	{
		xTail = x;
		yTail = y - len;
	}
	else
	{
		if ((angle > 0.0 && angle < 0.5*PI) || (angle > 1.5*PI && angle < 2.0*PI))
		{
			xTail = x + len / div;
		}
		else if ((angle > 0.5*PI && angle < PI) || (angle > PI && angle < 1.5*PI))
		{
			xTail = x - len / div;
		}
		yTail = y + dTanAng*(xTail - x);
	}

	return 1;
}

/////////////////////////////
///// How to use/////////////
/////////////////////////////
double angle = 60;
while (angle > 360)
	angle -= 360;
double xOri = 0, yOri = 0;
double xTail = 0.0, yTail = 0.0;
ComputeTail(xOri, yOri, angle * PI / 180, len, xTail, yTail);


```

# 72.  key byte 


```
__int64   MakeKey(int type, int code)
{
	return ((__int64)(type) << 32) + (code);
}

unsigned int DecodeFixed32(const char* ptr)
{
	return ((static_cast<unsigned int>(ptr[0]))
			| (static_cast<unsigned int>(ptr[1]) << 8)
			| (static_cast<unsigned int>(ptr[2]) << 16)
			| (static_cast<unsigned int>(ptr[3]) << 24));
}

```

# 73. func(param) = delete;

```

#include <cstdio>
class TestClass
{
public:
    void func(int data) { printf("data: %d\n", data); }
};
int main(void)
{
    TestClass obj;
    obj.func(100);
    obj.func(100.0); /// convert float to interger quietly;
    return 0;
}

//////////////////////////////

#include <cstdio>
class TestClass
{
public:
    void func(int data) { printf("data: %d\n", data); }
    void func(double data) = delete;  ////this function is forbidden;
};
int main(void)
{
    TestClass obj;
    obj.func(100);
    obj.func(100.0);  /// will complain " error: use of deleted function"  when compiling
    return 0;
}

```

# 74. compile pixman on windows

## 1.STEPS in cygwin
```
1. get pixman tar file 'pixman-0.40.0.tar.gz',open the folder in which contain target file;
2. uncompress the tar file above, 
   tar -zxvf  pixman-0.40.0.tar.gz
3. open cygwin env,cd pixman-0.40.0
4.configure -help ,get some detail about configure
  pay attention about '--host='  to set the target environment which match with the target os;
  if wanna depend on png library,please set 'PNG_fLAGS' and 'PNG_LIBS' 
5.when configure ok,then make and make install
6.search the builded header file and library file in the target folder which depend on the flag '--prefix' 
7.enjoy it;
```
## 2. INSTANCE in cygwin
```
Administrator@0812-P /cygdrive/f/cario/pixman-0.40.0/build
$ ../configure --prefix=/cygdrive/f/cario/cygwin/pixman --host=x86_64-w64-mingw32  PNG_CFLAGS='-I/cygdrive/f/cario/install/png/include' PNG_LIBS='-L/cygdrive/f/cario/install/png/lib -lpng16'
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /usr/bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking whether make supports nested variables... (cached) yes
checking build system type... x86_64-w64-mingw32
checking host system type... x86_64-w64-mingw32
checking for x86_64-w64-mingw32-gcc... x86_64-w64-mingw32-gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.exe
checking for suffix of executables... .exe
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether x86_64-w64-mingw32-gcc accepts -g... yes
checking for x86_64-w64-mingw32-gcc option to accept ISO C89... none needed
checking whether x86_64-w64-mingw32-gcc understands -c and -o together... yes
checking whether make supports the include directive... yes (GNU style)
checking dependency style of x86_64-w64-mingw32-gcc... gcc3
checking dependency style of x86_64-w64-mingw32-gcc... gcc3
checking how to print strings... printf
checking for a sed that does not truncate output... /usr/bin/sed
checking for grep that handles long lines and -e... /usr/bin/grep
checking for egrep... /usr/bin/grep -E
checking for fgrep... /usr/bin/grep -F
checking for ld used by x86_64-w64-mingw32-gcc... /usr/x86_64-w64-mingw32/bin/ld.exe
checking if the linker (/usr/x86_64-w64-mingw32/bin/ld.exe) is GNU ld... yes
checking for BSD- or MS-compatible name lister (nm)... /usr/bin/x86_64-w64-mingw32-nm -B
checking the name lister (/usr/bin/x86_64-w64-mingw32-nm -B) interface... BSD nm
checking whether ln -s works... yes
checking the maximum length of command line arguments... 8192
checking how to convert x86_64-w64-mingw32 file names to x86_64-w64-mingw32 format... func_convert_file_msys_to_w32
checking how to convert x86_64-w64-mingw32 file names to toolchain format... func_convert_file_msys_to_w32
checking for /usr/x86_64-w64-mingw32/bin/ld.exe option to reload object files... -r
checking for x86_64-w64-mingw32-objdump... x86_64-w64-mingw32-objdump
checking how to recognize dependent libraries... file_magic ^x86 archive import|^x86 DLL
checking for x86_64-w64-mingw32-dlltool... x86_64-w64-mingw32-dlltool
checking how to associate runtime and link libraries... func_cygming_dll_for_implib
checking for x86_64-w64-mingw32-ar... x86_64-w64-mingw32-ar
checking for archiver @FILE support... @
checking for x86_64-w64-mingw32-strip... x86_64-w64-mingw32-strip
checking for x86_64-w64-mingw32-ranlib... x86_64-w64-mingw32-ranlib
checking command to parse /usr/bin/x86_64-w64-mingw32-nm -B output from x86_64-w64-mingw32-gcc object... ok
checking for sysroot... no
checking for a working dd... /usr/bin/dd
checking how to truncate binary pipes... /usr/bin/dd bs=4096 count=1
checking for x86_64-w64-mingw32-mt... no
checking for mt... mt
checking if mt is a manifest tool... yes
checking how to run the C preprocessor... x86_64-w64-mingw32-gcc -E
checking for ANSI C header files... yes
checking for sys/types.h... yes
checking for sys/stat.h... yes
checking for stdlib.h... yes
checking for string.h... yes
checking for memory.h... yes
checking for strings.h... yes
checking for inttypes.h... yes
checking for stdint.h... yes
checking for unistd.h... yes
checking for dlfcn.h... no
checking for objdir... .libs
checking if x86_64-w64-mingw32-gcc supports -fno-rtti -fno-exceptions... no
checking for x86_64-w64-mingw32-gcc option to produce PIC... -DDLL_EXPORT -DPIC
checking if x86_64-w64-mingw32-gcc PIC flag -DDLL_EXPORT -DPIC works... yes
checking if x86_64-w64-mingw32-gcc static flag -static works... yes
checking if x86_64-w64-mingw32-gcc supports -c -o file.o... yes
checking if x86_64-w64-mingw32-gcc supports -c -o file.o... (cached) yes
checking whether the x86_64-w64-mingw32-gcc linker (/usr/x86_64-w64-mingw32/bin/ld.exe) supports shared libraries... yes
checking whether -lc should be explicitly linked in... yes
checking dynamic linker characteristics... Win32 ld.exe
checking how to hardcode library paths into programs... immediate
checking whether stripping libraries is possible... yes
checking if libtool supports shared libraries... yes
checking whether to build shared libraries... yes
checking whether to build static libraries... yes
checking for getisax... no
checking whether byte ordering is bigendian... no
checking for inline... inline
checking whether the compiler supports -Werror... yes
checking size of long... 4
checking whether __SUNPRO_C is declared... no
checking whether __amd64 is declared... yes
checking whether the compiler supports -Wall... yes
checking whether the compiler supports -Wdeclaration-after-statement... yes
checking whether the compiler supports -Wno-unused-local-typedefs... yes
checking whether the compiler supports -fno-strict-aliasing... yes
checking for x86_64-w64-mingw32-gcc option to support OpenMP... -fopenmp
checking whether the compiler supports -fvisibility=hidden... no
checking whether the compiler supports -xldscope=hidden... no
checking whether to use Loongson MMI assembler... no
checking whether to use MMX intrinsics... yes
checking whether to use SSE2 intrinsics... yes
checking whether to use SSSE3 intrinsics... yes
checking whether to use VMX/Altivec intrinsics... no
checking whether to use ARM SIMD assembler... no
checking whether to use ARM NEON assembler... no
checking whether to use ARM IWMMXT intrinsics... no
checking whether to use MIPS DSPr2 assembler... no
checking whether to use GNU-style inline assembler... yes
checking for x86_64-w64-mingw32-pkg-config... /usr/bin/x86_64-w64-mingw32-pkg-config
checking pkg-config is at least version 0.9.0... yes
checking for pixman_version_string in -lpixman-1... no
checking for posix_memalign... no
checking for sigaction... no
checking for alarm... yes
checking sys/mman.h usability... no
checking sys/mman.h presence... no
checking for sys/mman.h... no
checking for mmap... no
checking for mprotect... yes
checking for getpagesize... yes
checking fenv.h usability... yes
checking fenv.h presence... yes
checking for fenv.h... yes
checking for feenableexcept in -lm... no
checking whether FE_DIVBYZERO is declared... yes
checking for gettimeofday... yes
checking sys/time.h usability... yes
checking sys/time.h presence... yes
checking for sys/time.h... yes
checking for library containing sqrtf... none required
checking for thread local storage (TLS) support... __thread
checking for pthreads... yes
checking for __attribute__((constructor))... yes
checking for __float128... yes
checking for __builtin_clz... yes
checking for GCC vector extensions... yes
checking for PNG... yes
checking that generated files are newer than configure... done
configure: creating ./config.status
config.status: creating pixman-1.pc
config.status: creating pixman-1-uninstalled.pc
config.status: creating Makefile
config.status: creating pixman/Makefile
config.status: creating pixman/pixman-version.h
config.status: creating demos/Makefile
config.status: creating test/Makefile
config.status: creating config.h
config.status: executing depfiles commands
config.status: executing libtool commands
```


## 1.STEPS on X64 MSVC

```
1. get pixman tar file 'pixman-0.42.2.tar.gz',open the folder in which contain target file;
2. uncompress the tar file above, 
   tar -zxvf  pixman-0.42.2.tar.gz
3. NOTE: if u compile pixman with the instruction 'end to end build for win32' for cario,
   u will get the x86 static and dynamic library; 
   in order to get the  X64 library,we should use the correct environment;
  
   open visual studio tools 'VS2015 x64 command prompts';
   
4. in vs2015 x64 command prompts window, 
   cd $(DIR)\pixman-0.42.2\pixman
   
   sed s/-MD/-MT/ Makefile.win32 > Makefile.fixed
   move /Y Makefile.fixed Makefile.win32
   
   make -f Makefile.win32 "CFG=release" clean
   rm -f release/*.exe release/*.ilk release/*.lib release/*.obj release/*.pdb

   make -f Makefile.win32 "CFG=release"
   
5. then in folder $(DIR)\pixman-0.42.2\pixman\release , we can get the target lib 'pixman-1.lib';
   
6. check the library with dumpbin command;
   dumpbin -headers $(DIR)\pixman-0.42.2\pixman\release\pixman-1.lib

7. if counter following question:
   pixman-mmx.c(276): error C2440: “return”: 无法从“int”转换为“__m64”
   
   mmx do not work on X64 MSVC,so we should remove macro USE_X86_MMX;
   
   in file 'pixman-0.42.2\pixman\Makefile.win32', change 
   from 'MMX_CFLAGS = -DUSE_X86_MMX -w14710 -w14714'
   to 'MMX_CFLAGS =  -w14710 -w14714'
   

```

## 2. INSTANCE in  X64 MSVC
```
D:\Program Files\Microsoft Visual Studio 14.0\VC>set PATH=%PATH%;D:\program\msys64\usr\bin

D:\Program Files\Microsoft Visual Studio 14.0\VC>set INCLUDE=%INCLUDE%;E:\3rd\zlib\include;E:\3rd\libpng\include;E:\3rd\pixman\include;

D:\Program Files\Microsoft Visual Studio 14.0\VC>set INCLUDE=%INCLUDE%;E:\cairo\cairo-1.17.2\boilerplate;E:\cairo\cairo-1.17.2\src;

D:\Program Files\Microsoft Visual Studio 14.0\VC>set LIB=%LIB%;E:\3rd\zlib\lib\zlib.lib;E:\3rd\libpng\lib\libpng16.lib;E:\3rd\pixman\lib\pixman-1.lib;

D:\Program Files\Microsoft Visual Studio 14.0\VC>E:

E:\>cd E:\cairo\pixman-0.42.2\pixman

E:\cairo\pixman-0.42.2\pixman>sed s/-MD/-MT/ Makefile.win32 > Makefile.fixed

E:\cairo\pixman-0.42.2\pixman>move /Y Makefile.fixed Makefile.win32
移动了         1 个文件。

E:\cairo\pixman-0.42.2\pixman>make -f Makefile.win32 "CFG=release" clean
rm -f release/*.exe release/*.ilk release/*.lib release/*.obj release/*.pdb

E:\cairo\pixman-0.42.2\pixman>make -f Makefile.win32 "CFG=release"
Setting MMX flag to default value 'on'... (use MMX=on or MMX=off)
Setting SSE2 flag to default value 'on'... (use SSE2=on or SSE2=off)
Setting SSSE3 flag to default value 'on'... (use SSSE3=on or SSSE3=off)
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman.obj" pixman.c
pixman.c
e:\cairo\pixman-0.42.2\pixman\pixman.c(218) : warning C4710: “clip_general_image”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman.c(266) : warning C4710: “clip_general_image”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman.c(284) : warning C4710: “clip_general_image”: 函数未内联
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-access.obj" pixman-access.c
pixman-access.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-access-accessors.obj" pixman-access-accessors.c
pixman-access-accessors.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-bits-image.obj" pixman-bits-image.c
pixman-bits-image.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-combine32.obj" pixman-combine32.c
pixman-combine32.c
pixman-combine32.c: warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据 丢失
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-combine-float.obj" pixman-combine-float.c
pixman-combine-float.c
pixman-combine-float.c: warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止 数据丢失
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-conical-gradient.obj" pixman-conical-gradient.c
pixman-conical-gradient.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-filter.obj" pixman-filter.c
pixman-filter.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-x86.obj" pixman-x86.c
pixman-x86.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-mips.obj" pixman-mips.c
pixman-mips.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-arm.obj" pixman-arm.c
pixman-arm.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-ppc.obj" pixman-ppc.c
pixman-ppc.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-edge.obj" pixman-edge.c
pixman-edge.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-edge-accessors.obj" pixman-edge-accessors.c
pixman-edge-accessors.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-fast-path.obj" pixman-fast-path.c
pixman-fast-path.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-glyph.obj" pixman-glyph.c
pixman-glyph.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-general.obj" pixman-general.c
pixman-general.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-gradient-walker.obj" pixman-gradient-walker.c
pixman-gradient-walker.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-image.obj" pixman-image.c
pixman-image.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-implementation.obj" pixman-implementation.c
pixman-implementation.c
pixman-implementation.c(418): warning C4710: “int printf(const char *const ,...)”: 函数未内联
C:\Program Files (x86)\Windows Kits\10\include\10.0.10240.0\ucrt\stdio.h(944): note: 参见“printf”的声明
c:\program files (x86)\windows kits\10\include\10.0.10240.0\ucrt\stdio.h(639) : warning C4710: “__local_stdio_printf_options”: 函数未内联
E:\cairo\pixman-0.42.2\pixman\pixman-implementation.c : warning C4710: “__local_stdio_printf_options”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-implementation.c(371) : warning C4710: “printf”: 函数未内 联
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-linear-gradient.obj" pixman-linear-gradient.c
pixman-linear-gradient.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-matrix.obj" pixman-matrix.c
pixman-matrix.c
e:\cairo\pixman-0.42.2\pixman\pixman-matrix.c(279) : warning C4710: “rounded_sdiv_128_by_49”:  函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-matrix.c(283) : warning C4710: “rounded_sdiv_128_by_49”:  函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-matrix.c(294) : warning C4710: “rounded_sdiv_128_by_49”:  函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-matrix.c(298) : warning C4710: “rounded_sdiv_128_by_49”:  函数未内联
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-noop.obj" pixman-noop.c
pixman-noop.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-radial-gradient.obj" pixman-radial-gradient.c
pixman-radial-gradient.c
pixman-radial-gradient.c: warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防 止数据丢失
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-region16.obj" pixman-region16.c
pixman-region16.c
pixman-region16.c(68): warning C4710: “int fprintf(FILE *const ,const char *const ,...)”: 函数未内联
C:\Program Files (x86)\Windows Kits\10\include\10.0.10240.0\ucrt\stdio.h(824): note: 参见“fprintf”的声明
c:\program files (x86)\windows kits\10\include\10.0.10240.0\ucrt\stdio.h(639) : warning C4710: “__local_stdio_printf_options”: 函数未内联
E:\cairo\pixman-0.42.2\pixman\pixman-region16.c : warning C4710: “__local_stdio_printf_options”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(348) : warning C4710: “fprintf”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(349) : warning C4710: “fprintf”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(2681) : warning C4710: “bitmap_addrect”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(2715) : warning C4710: “bitmap_addrect”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(894) : warning C4710: “pixman_region_append_non_o ”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(896) : warning C4710: “pixman_coalesce”: 函数未内联
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-region32.obj" pixman-region32.c
pixman-region32.c
pixman-region32.c(48): warning C4710: “int fprintf(FILE *const ,const char *const ,...)”: 函数未内联
C:\Program Files (x86)\Windows Kits\10\include\10.0.10240.0\ucrt\stdio.h(824): note: 参见“fprintf”的声明
c:\program files (x86)\windows kits\10\include\10.0.10240.0\ucrt\stdio.h(639) : warning C4710: “__local_stdio_printf_options”: 函数未内联
E:\cairo\pixman-0.42.2\pixman\pixman-region32.c : warning C4710: “__local_stdio_printf_options”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(348) : warning C4710: “fprintf”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(349) : warning C4710: “fprintf”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(357) : warning C4710: “fprintf”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(361) : warning C4710: “fprintf”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(894) : warning C4710: “pixman_region_append_non_o ”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(896) : warning C4710: “pixman_coalesce”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(912) : warning C4710: “pixman_region_append_non_o ”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(915) : warning C4710: “pixman_coalesce”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(942) : warning C4710: “pixman_coalesce”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(973) : warning C4710: “pixman_region_append_non_o ”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(980) : warning C4710: “pixman_coalesce”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(992) : warning C4710: “pixman_region_append_non_o ”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(999) : warning C4710: “pixman_coalesce”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(1687) : warning C4710: “pixman_coalesce”: 函数未 内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(1751) : warning C4710: “pixman_coalesce”: 函数未 内联
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-solid-fill.obj" pixman-solid-fill.c
pixman-solid-fill.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-timer.obj" pixman-timer.c
pixman-timer.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-trap.obj" pixman-trap.c
pixman-trap.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-utils.obj" pixman-utils.c
pixman-utils.c
pixman-utils.c(331): warning C4710: “int fprintf(FILE *const ,const char *const ,...)”: 函数未内联
C:\Program Files (x86)\Windows Kits\10\include\10.0.10240.0\ucrt\stdio.h(824): note: 参见“fprintf”的声明
c:\program files (x86)\windows kits\10\include\10.0.10240.0\ucrt\stdio.h(639) : warning C4710: “__local_stdio_printf_options”: 函数未内联
E:\cairo\pixman-0.42.2\pixman\pixman-utils.c : warning C4710: “__local_stdio_printf_options”:  函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-utils.c(322) : warning C4710: “fprintf”: 函数未内联
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -DUSE_X86_MMX -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-mmx.obj" pixman-mmx.c
pixman-mmx.c
pixman-mmx.c(276): error C2440: “return”: 无法从“int”转换为“__m64”
pixman-mmx.c(278): error C2440: “return”: 无法从“int”转换为“__m64”
pixman-mmx.c(286): error C2440: “return”: 无法从“int”转换为“__m64”
pixman-mmx.c(317): error C2440: “=”: 无法从“int”转换为“__m64”
pixman-mmx.c(318): error C2440: “=”: 无法从“int”转换为“__m64”
pixman-mmx.c(319): error C2440: “=”: 无法从“int”转换为“__m64”
pixman-mmx.c(353): error C2440: “return”: 无法从“int”转换为“__m64”
pixman-mmx.c(360): error C2440: “初始化”: 无法从“int”转换为“__m64”
pixman-mmx.c(433): error C2440: “return”: 无法从“int”转换为“__m64”
pixman-mmx.c(443): error C2440: “return”: 无法从“int”转换为“__m64”
pixman-mmx.c(457): error C2440: “return”: 无法从“int”转换为“__m64”
pixman-mmx.c(720): error C2440: “=”: 无法从“int”转换为“__m64”
pixman-mmx.c(729): error C2440: “函数”: 无法从“int”转换为“__m64”
pixman-mmx.c(729): warning C4024: “over”: 形参和实参 3 的类型不同
pixman-mmx.c(732): error C2440: “return”: 无法从“int”转换为“__m64”
pixman-mmx.c(2419): error C2440: “函数”: 无法从“int”转换为“__m64”
pixman-mmx.c(2419): warning C4024: “pack_565”: 形参和实参 2 的类型不同
pixman-mmx.c(3970): error C2440: “初始化”: 无法从“int”转换为“__m64”
make: *** [../Makefile.win32.common:68: release/pixman-mmx.obj] Error 2

E:\cairo\pixman-0.42.2\pixman>make -f Makefile.win32 "CFG=release" clean
rm -f release/*.exe release/*.ilk release/*.lib release/*.obj release/*.pdb

E:\cairo\pixman-0.42.2\pixman>make -f Makefile.win32 "CFG=release"
Setting MMX flag to default value 'on'... (use MMX=on or MMX=off)
Setting SSE2 flag to default value 'on'... (use SSE2=on or SSE2=off)
Setting SSSE3 flag to default value 'on'... (use SSSE3=on or SSSE3=off)
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman.obj" pixman.c
pixman.c
e:\cairo\pixman-0.42.2\pixman\pixman.c(218) : warning C4710: “clip_general_image”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman.c(266) : warning C4710: “clip_general_image”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman.c(284) : warning C4710: “clip_general_image”: 函数未内联
E:\cairo\pixman-0.42.2\pixman\pixman.c : warning C4710: “clip_general_image”: 函数未内联
E:\cairo\pixman-0.42.2\pixman\pixman.c : warning C4710: “clip_general_image”: 函数未内联
E:\cairo\pixman-0.42.2\pixman\pixman.c : warning C4710: “clip_general_image”: 函数未内联
E:\cairo\pixman-0.42.2\pixman\pixman.c : warning C4710: “clip_general_image”: 函数未内联
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-access.obj" pixman-access.c
pixman-access.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-access-accessors.obj" pixman-access-accessors.c
pixman-access-accessors.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-bits-image.obj" pixman-bits-image.c
pixman-bits-image.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-combine32.obj" pixman-combine32.c
pixman-combine32.c
pixman-combine32.c: warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据 丢失
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-combine-float.obj" pixman-combine-float.c
pixman-combine-float.c
pixman-combine-float.c: warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止 数据丢失
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-conical-gradient.obj" pixman-conical-gradient.c
pixman-conical-gradient.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-filter.obj" pixman-filter.c
pixman-filter.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-x86.obj" pixman-x86.c
pixman-x86.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-mips.obj" pixman-mips.c
pixman-mips.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-arm.obj" pixman-arm.c
pixman-arm.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-ppc.obj" pixman-ppc.c
pixman-ppc.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-edge.obj" pixman-edge.c
pixman-edge.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-edge-accessors.obj" pixman-edge-accessors.c
pixman-edge-accessors.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-fast-path.obj" pixman-fast-path.c
pixman-fast-path.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-glyph.obj" pixman-glyph.c
pixman-glyph.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-general.obj" pixman-general.c
pixman-general.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-gradient-walker.obj" pixman-gradient-walker.c
pixman-gradient-walker.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-image.obj" pixman-image.c
pixman-image.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-implementation.obj" pixman-implementation.c
pixman-implementation.c
pixman-implementation.c(418): warning C4710: “int printf(const char *const ,...)”: 函数未内联
C:\Program Files (x86)\Windows Kits\10\include\10.0.10240.0\ucrt\stdio.h(944): note: 参见“printf”的声明
c:\program files (x86)\windows kits\10\include\10.0.10240.0\ucrt\stdio.h(639) : warning C4710: “__local_stdio_printf_options”: 函数未内联
E:\cairo\pixman-0.42.2\pixman\pixman-implementation.c : warning C4710: “__local_stdio_printf_options”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-implementation.c(371) : warning C4710: “printf”: 函数未内 联
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-linear-gradient.obj" pixman-linear-gradient.c
pixman-linear-gradient.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-matrix.obj" pixman-matrix.c
pixman-matrix.c
e:\cairo\pixman-0.42.2\pixman\pixman-matrix.c(279) : warning C4710: “rounded_sdiv_128_by_49”:  函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-matrix.c(283) : warning C4710: “rounded_sdiv_128_by_49”:  函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-matrix.c(294) : warning C4710: “rounded_sdiv_128_by_49”:  函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-matrix.c(298) : warning C4710: “rounded_sdiv_128_by_49”:  函数未内联
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-noop.obj" pixman-noop.c
pixman-noop.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-radial-gradient.obj" pixman-radial-gradient.c
pixman-radial-gradient.c
pixman-radial-gradient.c: warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防 止数据丢失
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-region16.obj" pixman-region16.c
pixman-region16.c
pixman-region16.c(68): warning C4710: “int fprintf(FILE *const ,const char *const ,...)”: 函数未内联
C:\Program Files (x86)\Windows Kits\10\include\10.0.10240.0\ucrt\stdio.h(824): note: 参见“fprintf”的声明
c:\program files (x86)\windows kits\10\include\10.0.10240.0\ucrt\stdio.h(639) : warning C4710: “__local_stdio_printf_options”: 函数未内联
E:\cairo\pixman-0.42.2\pixman\pixman-region16.c : warning C4710: “__local_stdio_printf_options”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(348) : warning C4710: “fprintf”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(349) : warning C4710: “fprintf”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(357) : warning C4710: “fprintf”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(361) : warning C4710: “fprintf”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(2681) : warning C4710: “bitmap_addrect”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(2715) : warning C4710: “bitmap_addrect”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(894) : warning C4710: “pixman_region_append_non_o ”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(896) : warning C4710: “pixman_coalesce”: 函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-region.c(999) : warning C4710: “pixman_coalesce”: 函数未内联
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-region32.obj" pixman-region32.c
pixman-region32.c
pixman-region32.c(48): warning C4710: “int fprintf(FILE *const ,const char *const ,...)”: 函数未内联
C:\Program Files (x86)\Windows Kits\10\include\10.0.10240.0\ucrt\stdio.h(824): note: 参见“fprintf”的声明
c:\program files (x86)\windows kits\10\include\10.0.10240.0\ucrt\stdio.h(639) : warning C4710: “__local_stdio_printf_options”: 函数未内联
E:\cairo\pixman-0.42.2\pixman\pixman-region32.c : warning C4710: “__local_stdio_printf_options”: 函数未内联
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-solid-fill.obj" pixman-solid-fill.c
pixman-solid-fill.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-timer.obj" pixman-timer.c
pixman-timer.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-trap.obj" pixman-trap.c
pixman-trap.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-utils.obj" pixman-utils.c
pixman-utils.c
pixman-utils.c(331): warning C4710: “int fprintf(FILE *const ,const char *const ,...)”: 函数未内联
C:\Program Files (x86)\Windows Kits\10\include\10.0.10240.0\ucrt\stdio.h(824): note: 参见“fprintf”的声明
c:\program files (x86)\windows kits\10\include\10.0.10240.0\ucrt\stdio.h(639) : warning C4710: “__local_stdio_printf_options”: 函数未内联
E:\cairo\pixman-0.42.2\pixman\pixman-utils.c : warning C4710: “__local_stdio_printf_options”:  函数未内联
e:\cairo\pixman-0.42.2\pixman\pixman-utils.c(322) : warning C4710: “fprintf”: 函数未内联
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-mmx.obj" pixman-mmx.c
pixman-mmx.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-sse2.obj" pixman-sse2.c
pixman-sse2.c
cl -c -nologo -I. -I.. -I../pixman -DPACKAGE=pixman-1 -DPACKAGE_VERSION="" -DPACKAGE_BUGREPORT="" -MD -O2  -w14710 -w14714 -DUSE_SSE2 -DUSE_SSSE3 -Fo"release/pixman-ssse3.obj" pixman-ssse3.c
pixman-ssse3.c

E:\cairo\pixman-0.42.2\pixman>dumpbin -headers E:\cairo\pixman-0.42.2\pixman\release\pixman-1.lib
Microsoft (R) COFF/PE Dumper Version 14.00.24210.0
Copyright (C) Microsoft Corporation.  All rights reserved.

Dump of file E:\cairo\pixman-0.42.2\pixman\release\pixman-1.lib

File Type: LIBRARY

FILE HEADER VALUES
            8664 machine (x64)
              14 number of sections
        63ABB780 time date stamp Wed Dec 28 11:26:56 2022
             B31 file pointer to symbol table
              47 number of symbols
               0 size of optional header
               0 characteristics

SECTION HEADER #1
.drectve name
       0 physical address
       0 virtual address
      2F size of raw data
     334 file pointer to raw data (00000334 to 00000362)
       0 file pointer to relocation table
       0 file pointer to line numbers
       0 number of relocations
       0 number of line numbers
  100A00 flags
         Info
         Remove
         1 byte align

E:\cairo\pixman-0.42.2\pixman>cd ../../

```


# 75 .build cario on windows

## 1.STEPS in CYGWIN
```
1. get cario tar file 'cairo-1.16.0.tar.xz',open the folder in which contain target file;
2. uncompress the tar file above, 
   tar -zxvf  cairo-1.16.0.tar.xz
3. open cygwin env,cd cairo-1.16.0
4.configure -help ,get some detail about configure
  pay attention about '--host='  to set the target environment which match with the target os;
  if wanna depend on png library,please set 'png_CFLAGS' and 'png_LIBS' ;
  if counter error: 'configure: error: recommended script surface backend feature could not be enabled,
  checking whether cairo's script surface backend feature could be enabled... no (requires zlib http://www.gzip.org/zlib/)',
5.when configure ok,then make and make install
6.search the builded header file and library file in the target folder which depend on the flag '--prefix' 
7.enjoy it;
```
## 2. INSTANCE in CYGWIN
```
Administrator@YR170812-LMCY /cygdrive/f/cario/cairo-1.16.0/bd
$  ../configure --prefix=/cygdrive/f/cario/install/cario --host=x86_64-w64-mingw32  png_CFLAGS='-I/cygdrive/f/cario/install/png/include' png_LIBS='-L/cygdrive/f/cario/install/png/lib  -lpng16' png_REQUIRES='libpng16' pixman_CFLAGS='-I/cygdrive/f/cario/install/pixman/include/pixman-1' pixman_LIBS='-L/cygdrive/f/cario/install/pixman/lib  -lpixman-1'  CFLAGS='-I/cygdrive/f/cario/install/zlib/include' LIBS='-L/cygdrive/f/cario/install/zlib/lib -lz' --enable-ft=no

checking for x86_64-w64-mingw32-gcc... x86_64-w64-mingw32-gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.exe
checking for suffix of executables... .exe
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether x86_64-w64-mingw32-gcc accepts -g... yes
checking for x86_64-w64-mingw32-gcc option to accept ISO C89... none needed
checking whether x86_64-w64-mingw32-gcc understands -c and -o together... yes
checking how to run the C preprocessor... x86_64-w64-mingw32-gcc -E

checking whether make sets $(MAKE)... yes
checking for style of include used by make... GNU
checking whether make supports nested variables... yes
checking dependency style of x86_64-w64-mingw32-gcc... gcc3
checking whether make supports nested variables... (cached) yes
checking for x86_64-w64-mingw32-ar... x86_64-w64-mingw32-ar
checking the archiver (x86_64-w64-mingw32-ar) interface... ar
checking build system type... x86_64-unknown-cygwin
checking host system type... x86_64-w64-mingw32
checking how to print strings... printf
checking for a sed that does not truncate output... /usr/bin/sed
checking for fgrep... /usr/bin/grep -F
checking for ld used by x86_64-w64-mingw32-gcc... /usr/x86_64-w64-mingw32/bin/ld.exe
checking if the linker (/usr/x86_64-w64-mingw32/bin/ld.exe) is GNU ld... yes
checking for BSD- or MS-compatible name lister (nm)... /usr/bin/x86_64-w64-mingw32-nm -B
checking the name lister (/usr/bin/x86_64-w64-mingw32-nm -B) interface... BSD nm
checking whether ln -s works... yes
checking the maximum length of command line arguments... 8192
checking how to convert x86_64-unknown-cygwin file names to x86_64-w64-mingw32 format... func_convert_file_cygwin_to_w32
checking how to convert x86_64-unknown-cygwin file names to toolchain format... func_convert_file_noop
checking for /usr/x86_64-w64-mingw32/bin/ld.exe option to reload object files... -r
checking for x86_64-w64-mingw32-objdump... x86_64-w64-mingw32-objdump
checking how to recognize dependent libraries... file_magic ^x86 archive import|^x86 DLL
checking for x86_64-w64-mingw32-dlltool... x86_64-w64-mingw32-dlltool
checking how to associate runtime and link libraries... func_cygming_dll_for_implib
checking for x86_64-w64-mingw32-ar... (cached) x86_64-w64-mingw32-ar

checking whether stripping libraries is possible... yes
checking if libtool supports shared libraries... yes
checking whether to build shared libraries... yes
checking whether to build static libraries... yes
checking for x86_64-w64-mingw32-pkg-config... /usr/bin/x86_64-w64-mingw32-pkg-config
checking pkg-config is at least version 0.9.0... yes
checking for gtk-doc... no

checking whether x86_64-w64-mingw32-gcc supports -Wunused-but-set-variable -Wno-unused-but-set-variable... yes
checking which warning flags were supported...  -Wall -Wextra -Wmissing-declarations -Werror-implicit-function-declaration -Wpointer-arith -Wwrite-strings -Wsign-compare -Wpacked -Wswitch-enum -Wmissing-format-attribute -Wvolatile-register-var -Wstrict-aliasing=2 -Winit-self -Wunsafe-loop-optimizations -Wno-missing-field-initializers -Wno-unused-parameter -Wno-attributes -Wno-long-long -Winline -fno-strict-aliasing -fno-common -Wp,-D_FORTIFY_SOURCE=2 -Wno-unused-but-set-variable
checking how to enable unused result warnings... __attribute__((__warn_unused_result__))
checking how to allow undefined symbols in shared libraries used by test suite... -Wl,--allow-shlib-undefined
checking whether byte ordering is bigendian... no
checking whether float word ordering is bigendian... no
checking for native atomic primitives... cxx11
checking whether atomic ops require a memory barrier... no
checking size of void *... 8
checking size of int... 4
checking size of long... 4
checking size of long long... 8
checking size of size_t... 8

checking for VALGRIND... no
no
checking for compress in -lz... yes
checking zlib.h usability... yes
checking zlib.h presence... no
configure: WARNING: zlib.h: accepted by the compiler, rejected by the preprocessor!
configure: WARNING: zlib.h: proceeding with the compiler's result
checking for zlib.h... yes
checking for lzo2a_decompress in -llzo2... no
checking for dlsym in -ldl... no
checking for dlsym... no

checking whether cairo's Xlib surface backend feature could be enabled... no (requires X development libraries)
checking for cairo's Xlib Xrender surface backend feature...
checking whether cairo's Xlib Xrender surface backend feature could be enabled... no (requires --enable-xlib)
checking for cairo's XCB surface backend feature...
checking for xcb... no
checking whether cairo's XCB surface backend feature could be enabled... no (requires xcb >= 1.6 xcb-render >= 1.6 https://xcb.freedesktop.org)
checking for cairo's XCB/SHM functions feature...
checking whether cairo's XCB/SHM functions feature could be enabled... no (requires --enable-xcb)
checking for cairo's Quartz surface backend feature...

checking for cairo's Microsoft Windows font backend feature...
checking whether cairo's Microsoft Windows font backend feature could be enabled... yes
checking for gs... no
configure: WARNING: Win32 Printing backend will not be tested since ghostscript is not available
checking for cairo's PNG functions feature...
checking for png... yes
checking whether cairo's PNG functions feature could be enabled... yes
checking for cairo's EGL functions feature...
checking whether cairo's EGL functions feature could be enabled... no (not required by any backend)
checking for cairo's GLX functions feature...
checking whether cairo's GLX functions feature could be enabled... no (not required by any backend)
checking for cairo's WGL functions feature...
checking whether cairo's WGL functions feature could be enabled... no (not required by any backend)
checking for cairo's script surface backend feature...
checking whether cairo's script surface backend feature could be enabled... yes

checking for FT_HAS_COLOR... yes
checking for cairo's PostScript surface backend feature...
checking whether cairo's PostScript surface backend feature could be enabled... yes
checking for gs... no
configure: WARNING: PS backend will not be tested since ghostscript is not available
checking for LIBSPECTRE... no
checking for cairo's PDF surface backend feature...
checking whether cairo's PDF surface backend feature could be enabled... yes
checking for POPPLER... no
configure: WARNING: PDF backend will not be tested since poppler >= 0.17.4 is not available
checking for cairo's SVG surface backend feature...
checking whether cairo's SVG surface backend feature could be enabled... yes
checking for LIBRSVG... no
configure: WARNING: SVG backend will not be tested since librsvg >= 2.35.0 is not available
checking for cairo's image surface backend feature...
checking for pixman... yes
checking whether cairo's image surface backend feature could be enabled... yes
checking for cairo's mime surface backend feature...
checking whether cairo's mime surface backend feature could be enabled... yes
checking for cairo's recording surface backend feature...
checking whether cairo's recording surface backend feature could be enabled... yes
checking for cairo's observer surface backend feature...
checking whether cairo's observer surface backend feature could be enabled... yes
checking for cairo's user font backend feature...
checking whether cairo's user font backend feature could be enabled... yes
checking for cairo's pthread feature...
checking whether cairo's pthread feature could be enabled... yes

checking that generated files are newer than configure... done
configure: creating ./config.status
config.status: creating src/cairo.pc
config.status: creating cairo-uninstalled.pc
config.status: creating src/cairo-win32.pc
config.status: creating cairo-win32-uninstalled.pc
config.status: creating src/cairo-win32-font.pc
config.status: creating cairo-win32-font-uninstalled.pc
config.status: creating src/cairo-png.pc
config.status: creating cairo-png-uninstalled.pc
config.status: creating src/cairo-script.pc
config.status: creating cairo-script-uninstalled.pc
config.status: creating src/cairo-ft.pc
config.status: creating cairo-ft-uninstalled.pc
config.status: creating src/cairo-ps.pc
config.status: creating cairo-ps-uninstalled.pc
config.status: creating src/cairo-pdf.pc
config.status: creating cairo-pdf-uninstalled.pc
config.status: creating src/cairo-svg.pc
config.status: creating cairo-svg-uninstalled.pc
config.status: creating Makefile
config.status: creating boilerplate/Makefile
config.status: creating src/Makefile
config.status: creating test/Makefile
config.status: creating test/pdiff/Makefile
config.status: creating perf/Makefile
config.status: creating perf/micro/Makefile
config.status: creating util/Makefile
config.status: creating util/cairo-fdr/Makefile
config.status: creating util/cairo-gobject/Makefile
config.status: creating util/cairo-missing/Makefile
config.status: creating util/cairo-script/Makefile
config.status: creating util/cairo-script/examples/Makefile
config.status: creating util/cairo-sphinx/Makefile
config.status: creating util/cairo-trace/Makefile
config.status: creating util/cairo-trace/cairo-trace
config.status: creating doc/Makefile
config.status: creating doc/public/Makefile
config.status: creating config.h
config.status: executing depfiles commands
config.status: executing libtool commands
config.status: executing ../build/Makefile.win32.features commands
config.status: creating ../build/Makefile.win32.features
config.status: ../build/Makefile.win32.features is unchanged
config.status: executing ../src/Makefile.am.features commands
config.status: creating ../src/Makefile.am.features
config.status: ../src/Makefile.am.features is unchanged
config.status: executing ../src/Makefile.win32.features commands
config.status: creating ../src/Makefile.win32.features
config.status: ../src/Makefile.win32.features is unchanged
config.status: executing ../boilerplate/Makefile.am.features commands
config.status: creating ../boilerplate/Makefile.am.features
config.status: ../boilerplate/Makefile.am.features is unchanged
config.status: executing ../boilerplate/Makefile.win32.features commands
config.status: creating ../boilerplate/Makefile.win32.features
config.status: ../boilerplate/Makefile.win32.features is unchanged
config.status: executing src/cairo-features.h commands
config.status: creating src/cairo-features.h
config.status: src/cairo-features.h is unchanged
config.status: executing src/cairo-supported-features.h commands
config.status: creating src/cairo-supported-features.h
config.status: src/cairo-supported-features.h is unchanged
config.status: executing ../build/Makefile.win32.features-h commands
config.status: creating ../build/Makefile.win32.features-h
config.status: ../build/Makefile.win32.features-h is unchanged
config.status: executing cairo-trace commands

cairo (version 1.16.0 [release]) will be compiled with:

The following surface backends:
  Image:         yes (always builtin)
  Recording:     yes (always builtin)
  Observer:      yes (always builtin)
  Mime:          yes (always builtin)
  Tee:           no (disabled, use --enable-tee to enable)
  XML:           no (disabled, use --enable-xml to enable)
  Xlib:          no (requires X development libraries)
  Xlib Xrender:  no (requires --enable-xlib)
  Qt:            no (disabled, use --enable-qt to enable)
  Quartz:        no (requires CoreGraphics framework)
  Quartz-image:  no (disabled, use --enable-quartz-image to enable)
  XCB:           no (requires xcb >= 1.6 xcb-render >= 1.6 https://xcb.freedesktop.org)
  Win32:         yes
  OS2:           no (disabled, use --enable-os2 to enable)
  CairoScript:   yes
  PostScript:    yes
  PDF:           yes
  SVG:           yes
  OpenGL:        no (disabled, use --enable-gl to enable)
  OpenGL ES 2.0: no (disabled, use --enable-glesv2 to enable)
  OpenGL ES 3.0: no (disabled, use --enable-glesv3 to enable)
  BeOS:          no (disabled, use --enable-beos to enable)
  DirectFB:      no (disabled, use --enable-directfb to enable)
  OpenVG:        no (disabled, use --enable-vg to enable)
  DRM:           no (disabled, use --enable-drm to enable)
  Cogl:          no (disabled, use --enable-cogl to enable)

The following font backends:
  User:          yes (always builtin)
  FreeType:      no (disabled, use --enable-ft to enable)
  Fontconfig:    no (disabled, use --enable-ft to enable)
  Win32:         yes
  Quartz:        no (requires CoreGraphics framework)

The following functions:
  PNG functions:   yes
  GLX functions:   no (not required by any backend)
  WGL functions:   no (not required by any backend)
  EGL functions:   no (not required by any backend)
  X11-xcb functions: no (disabled, use --enable-xlib-xcb to enable)
  XCB-shm functions: no (requires --enable-xcb)

The following features and utilities:
  cairo-trace:                no (requires dynamic linker and zlib and real pthreads)
  cairo-script-interpreter:   yes

And the following internal features:
  pthread:       yes
  gtk-doc:       no
  gcov support:  no
  symbol-lookup: no (requires bfd)
  test surfaces: no (disabled, use --enable-test-surfaces to enable)
  ps testing:    no (requires libspectre)
  pdf testing:   no (requires poppler-glib >= 0.17.4)
  svg testing:   no (requires librsvg-2.0 >= 2.35.0)
  win32 printing testing:    no (requires ghostscript)



```

## compile in msys64
```
../configure --prefix=/e/cairo/pixman --host=x86_64-pc-msys  --enable-shared --enable-static   --enable-libpng PNG_CFLAGS='-I/e/xspace/3rd/libpng/include' PNG_LIBS='-L/e/xspace/3rd/libpng/lib  -lpng16'

export LIBPNG_PATH=/e/xspace/3rd/libpng
export ZLIB_PATH=/e/xspace/3rd/zlib
export PIXMAN_PATH=/e/xspace/3rd/pixman

cd $(ROOTDIR)/cario
mkdir build_cario
cd build_cario

../configure --prefix=/e/cairo/cario --host=x86_64-pc-msys  --enable-shared --enable-static  pixman_CFLAGS='-I/e/xspace/3rd/pixman/include' pixman_LIBS='-L/e/xspace/3rd/pixman/lib -lpixman-1'  png_CFLAGS='-I/e/xspace/3rd/libpng/include' png_LIBS='-L/e/xspace/3rd/libpng/lib  -lpng16' png_REQUIRES='libpng16' CFLAGS='-I/e/xspace/3rd/zlib/include' LIBS='-L/e/xspace/3rd/zlib/lib -lz' --enable-ft=yes  FREETYPE_CFLAGS='-I/e/xspace/3rd/freetype/include' FREETYPE_LIBS='-L/e/xspace/3rd/freetype/lib -lfreetype' --enable-qt=yes  qt_CFLAGS='-I/e/xspace/3rd/Qt5.12.6/include' qt_LIBS='-L/e/xspace/3rd/Qt5.12.6/lib -lQt5Core'

make -j4


```


## 1.STEPS on X64 MSVC

```
1. get cairo tar file 'cairo-1.17.2.tar.gz',open the folder in which contain target file;
2. uncompress the tar file above, 
   tar -zxvf  cairo-1.17.2.tar.gz
3. NOTE: if u compile cairo with the instruction 'end to end build for win32' for cario,
   u will get the x86 static and dynamic library; 
   in order to get the  X64 library,we should use the correct environment;
  
   open visual studio tools 'VS2015 x64 command prompts';
   
4. in vs2015 x64 command prompts window, 
   
   [4.1] firstly,set the environment
   
   [4.1.0]
   set PATH=%PATH%;E:\msys64\usr\bin
   
   [4.1.1]
   set INCLUDE=%INCLUDE%;E:\3rd\zlib\include;E:\3rd\libpng\include;E:\3rd\pixman\include;
   set INCLUDE=%INCLUDE%;E:\cairo\cairo-1.17.2\boilerplate;E:\cairo\cairo-1.17.2\src;
   set LIB=%LIB%;E:\3rd\zlib\lib\zlib.lib;E:\3rd\libpng\lib\libpng16.lib;E:\3rd\pixman\lib\pixman-1.lib;

   [4.1.2]
   set LIBPNG_PATH=E:\3rd\libpng
   set ZLIB_PATH=E:\3rd\zlib
   set PIXMAN_PATH=E:\3rd\pixman

   [4.2]
   
   cd $(DIR)\cairo-1.17.2
   sed s/-MD/ build\Makefile.win32.common > build\Makefile.fixed
   move /Y build\Makefile.fixed build\Makefile.win32.common
   
   [4.3]
   make -f Makefile.win32 "CFG=release" clean
   make -f Makefile.win32 "CFG=release"
   
5. then in folder $(DIR)\cairo-1.17.2/src/release/ , we can get the target lib 'cairo.dll';
   
6. check the library with dumpbin command;
   dumpbin -headers $(DIR)\cairo1.17.2\lib\cairo.dll

7. if counter following question:
   [7.1] $(DIR)\cairo-1.17.2\src\cairoint.h(71): fatal error C1083: 无法打开包括文件: “pixman.h”: No such file or directory
   firstly set the environment,run steps 4 again;   [4.1.1] is important;
   
   
   [7.2] LINK : fatal error LNK1181: 无法打开输入文件“../../libpng/lib/libpng16.lib”
   in file '$(DIR)cairo-1.17.2\build\Makefile.win32.common', check the linked libraries；[4.1.2] is importtant；
   maybe we should modify some library path in order to find the target library;
   
   [7.3] if cannot find make, please set make.exe into PATH;   [4.1.0] is important;
   
8. support for the freetype font backend
   enable-ft  CAIRO_HAS_FT_FONT
	
	[8.1] in order to find some header files in freetype,please set following environment; 
	
	set INCLUDE=%INCLUDE%;$(DIR)\freetype\include;
    set LIB=%LIB%;$(DIR)\freetype\lib\freetype.lib;
	
    [8.2] Edit build/Makefile.win32.features to enable features to build
    change from ‘CAIRO_HAS_FT_FONT=0’ to 'CAIRO_HAS_FT_FONT=1'
	
	[8.3] Edit build/Makefile.win32.common for customization
	open file：$(DIR)/build/Makefile.win32.common, add followings:
	
	ifeq ($(FREETYPE_PATH),)
	FREETYPE_PATH := $(top_builddir)/../freetype
	endif
	FREETYPE_CFLAGS := -I$(FREETYPE_PATH)/include/
	FREETYPE_LIBS := $(FREETYPE_PATH)/lib/libfreetype.lib
	CAIRO_LIBS += $(FREETYPE_LIBS)
	
	DEFAULT_CFLAGS = -nologo $(CFG_CFLAGS)
	DEFAULT_CFLAGS += -I. -I$(top_srcdir) -I$(top_srcdir)/src
	DEFAULT_CFLAGS += $(PIXMAN_CFLAGS) $(LIBPNG_CFLAGS) $(ZLIB_CFLAGS) $(FREETYPE_CFLAGS)
	
	save the changes; then make again;

9. for error LNK2001:无法解析的外部符号__imp_strstr
   remove /-MT ,use /-MD only;
```

## 2. INSTANCE in  X64 MSVC

```
E:\cairo>cd cairo-1.17.2

E:\cairo\cairo-1.17.2>ls
AUTHORS           ChangeLog           ChangeLog.pre-1.4  Makefile.in     acinclude.m4     config.log    test
BIBLIOGRAPHY      ChangeLog.pre-1.0   ChangeLog.pre-1.6  Makefile.win32  aclocal.m4       configure     util
BUGS              ChangeLog.pre-1.10  ChangeLog.pre-1.8  NEWS            autogen.sh       configure.ac
CODING_STYLE      ChangeLog.pre-1.12  HACKING            PORTING_GUIDE   boilerplate      dir
COPYING           ChangeLog.pre-1.14  INSTALL            README          build            doc
COPYING-LGPL-2.1  ChangeLog.pre-1.16  KNOWN_ISSUES       README.win32    cairo-version.h  perf
COPYING-MPL-1.1   ChangeLog.pre-1.2   Makefile.am        RELEASING       config.h.in      src

E:\cairo\cairo-1.17.2>sed s/-MD/-MT/ build\Makefile.win32.common > build\Makefile.fixed

E:\cairo\cairo-1.17.2>move /Y build\Makefile.fixed build\Makefile.win32.common
移动了         1 个文件。

E:\cairo\cairo-1.17.2>
E:\cairo\cairo-1.17.2>make -f Makefile.win32 "CFG=release" clean
make[1]: Entering directory '/e/cairo/cairo-1.17.2/boilerplate'
make[1]: Leaving directory '/e/cairo/cairo-1.17.2/boilerplate'
make[1]: Entering directory '/e/cairo/cairo-1.17.2/perf'
make[1]: Leaving directory '/e/cairo/cairo-1.17.2/perf'
make[1]: Entering directory '/e/cairo/cairo-1.17.2/src'
make[1]: Leaving directory '/e/cairo/cairo-1.17.2/src'
make[1]: Entering directory '/e/cairo/cairo-1.17.2/test'
make[1]: Leaving directory '/e/cairo/cairo-1.17.2/test'

E:\cairo\cairo-1.17.2>make -f Makefile.win32 "CFG=release"

make[1]: Entering directory '/e/cairo/cairo-1.17.2/src'

cairo-analysis-surface.c
cairo-arc.c
cairo-array.c
cairo-atomic.c
cairo-base64-stream.c
cairo-base85-stream.c
cairo-bentley-ottmann-rectangular.c
cairo-bentley-ottmann-rectilinear.c
cairo-bentley-ottmann.c
cairo-bentley-ottmann.c: warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失
cairo-botor-scan-converter.c
cairo-botor-scan-converter.c: warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式 以防止数据丢失
cairo-boxes-intersect.c
cairo-boxes.c
cairo-cache.c
cairo-clip-boxes.c
cairo-clip-polygon.c
cairo-clip-region.c
cairo-clip-surface.c
cairo-clip-tor-scan-converter.c
cairo-clip.c
cairo-color.c
cairo-composite-rectangles.c
cairo-compositor.c
cairo-contour.c
cairo-damage.c
cairo-debug.c
cairo-default-context.c
cairo-device.c
cairo-error.c
cairo-fallback-compositor.c
cairo-fixed.c
cairo-font-face-twin-data.c
cairo-font-face-twin.c
cairo-font-face.c
cairo-font-options.c
cairo-freed-pool.c
cairo-freelist.c
cairo-gstate.c
cairo-hash.c
cairo-hull.c
cairo-image-compositor.c
cairo-image-info.c
cairo-image-source.c
cairo-image-surface.c
cairo-line.c
cairo-line.c: warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失
cairo-lzw.c
cairo-mask-compositor.c
cairo-matrix.c
cairo-mempool.c
cairo-mempool.c(292): warning C4311: “类型转换”: 从“void *”到“unsigned long”的指针截断
cairo-mempool.c(299): warning C4311: “类型转换”: 从“void *”到“unsigned long”的指针截断
cairo-mesh-pattern-rasterizer.c
cairo-misc.c
cairo-mono-scan-converter.c
cairo-mutex.c
cairo-no-compositor.c
cairo-observer.c
cairo-output-stream.c
cairo-paginated-surface.c
cairo-path-bounds.c
cairo-path-fill.c
cairo-path-fixed.c
cairo-path-in-fill.c
cairo-path-in-fill.c: warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数 据丢失
cairo-path-stroke-boxes.c
cairo-path-stroke-boxes.c: warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失
cairo-path-stroke-polygon.c
cairo-path-stroke-traps.c
cairo-path-stroke-tristrip.c
cairo-path-stroke.c
cairo-path.c
cairo-pattern.c
cairo-pen.c
cairo-pen.c: warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失
cairo-polygon-intersect.c
cairo-polygon-intersect.c: warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失
cairo-polygon-reduce.c
cairo-polygon-reduce.c: warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止 数据丢失
cairo-polygon.c
cairo-raster-source-pattern.c
cairo-recording-surface.c
cairo-rectangle.c
cairo-rectangular-scan-converter.c
cairo-region.c
cairo-rtree.c
cairo-scaled-font.c
cairo-scaled-font.c(639): warning C4311: “类型转换”: 从“cairo_font_face_t *”到“unsigned long”的指针截断
cairo-scaled-font.c(2882): warning C4311: “类型转换”: 从“cairo_scaled_font_t *”到“unsigned long”的指针截断
cairo-shape-mask-compositor.c
cairo-slope.c
cairo-spans-compositor.c
cairo-spans.c
cairo-spline.c
cairo-spline.c: warning C4819: 该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失
cairo-stroke-dash.c
cairo-stroke-style.c
cairo-surface-clipper.c
cairo-surface-fallback.c
cairo-surface-observer.c
cairo-surface-offset.c
cairo-surface-snapshot.c
cairo-surface-subsurface.c
cairo-surface-wrapper.c
cairo-surface.c
cairo-time.c
cairo-tor-scan-converter.c
cairo-tor22-scan-converter.c
cairo-toy-font-face.c
cairo-traps-compositor.c
cairo-traps.c
cairo-tristrip.c
cairo-unicode.c
cairo-user-font.c
cairo-version.c
cairo-wideint.c
cairo.c
cairo-cff-subset.c
cairo-scaled-font-subsets.c
cairo-scaled-font-subsets.c(258): warning C4311: “类型转换”: 从“cairo_scaled_font_t *”到“unsigned long”的指针截断
cairo-scaled-font-subsets.c(263): warning C4311: “类型转换”: 从“cairo_font_face_t *”到“unsigned long”的指针截断
cairo-truetype-subset.c
cairo-type1-fallback.c
cairo-type1-glyph-names.c
cairo-type1-subset.c
cairo-type3-glyph-surface.c
cairo-pdf-operators.c
cairo-pdf-shading.c
cairo-tag-attributes.c
cairo-deflate-stream.c
cairo-png.c
cairo-script-surface.c
cairo-script-surface.c(3050): warning C4312: “类型转换”: 从“unsigned long”转换到更大的“void *”
cairo-script-surface.c(3098): warning C4312: “类型转换”: 从“unsigned long”转换到更大的“void *”
cairo-script-surface.c(3398): warning C4311: “类型转换”: 从“void *”到“unsigned long”的指针截断
cairo-script-surface.c(3469): warning C4311: “类型转换”: 从“void *”到“unsigned long”的指针截断
cairo-script-surface.c(3478): warning C4311: “类型转换”: 从“void *”到“unsigned long”的指针截断
cairo-ps-surface.c
cairo-pdf-surface.c
cairo-pdf-interchange.c
cairo-tag-stack.c
cairo-svg-surface.c

LINK : fatal error LNK1181: 无法打开输入文件“../../libpng/lib/libpng16.lib”
make[1]: *** [Makefile.win32:16: release/cairo.dll] Error 157
make[1]: Leaving directory '/e/cairo/cairo-1.17.2/src'
make: *** [Makefile.win32:12: cairo] Error 2

E:\cairo\cairo-1.17.2>
E:\cairo\cairo-1.17.2>set LIB=%LIB%;E:\3rd\zlib\lib\zlib.lib;E:\3rd\libpng\lib\libpng16.lib;E:\3rd\pixman\lib\libpixman-1.lib;

E:\cairo\cairo-1.17.2>make -f Makefile.win32 "CFG=release"

make[1]: Entering directory '/e/cairo/cairo-1.17.2/src'

LINK : fatal error LNK1181: 无法打开输入文件“../../libpng/lib/libpng16.lib”
make[1]: *** [Makefile.win32:16: release/cairo.dll] Error 157
make[1]: Leaving directory '/e/cairo/cairo-1.17.2/src'
make: *** [Makefile.win32:12: cairo] Error 2


E:\cairo\cairo-1.17.2>echo %LIBPNG_PATH%
%LIBPNG_PATH%

E:\cairo\cairo-1.17.2>set LIBPNG_PATH=E:\3rd\libpng

E:\cairo\cairo-1.17.2>set ZLIB_PATH=E:\3rd\zlib

E:\cairo\cairo-1.17.2>set PIXMAN_PATH=E:\3rd\pixman

E:\cairo\cairo-1.17.2>echo %LIBPNG_PATH%
E:\3rd\libpng

E:\cairo\cairo-1.17.2>make -f Makefile.win32 "CFG=release"

make[1]: Entering directory '/e/cairo/cairo-1.17.2/src'

  正在创建库 release/cairo.lib 和对象 release/cairo.exp
LINK : warning LNK4098: 默认库“MSVCRT”与其他库的使用冲突；请使用 /NODEFAULTLIB:library
libpixman-1.lib(pixman-access-accessors.obj) : warning LNK4049: 已导入本地定义的符号 free
libpixman-1.lib(pixman-conical-gradient.obj) : warning LNK4049: 已导入本地定义的符号 free
libpixman-1.lib(pixman-ssse3.obj) : warning LNK4049: 已导入本地定义的符号 free
libpixman-1.lib(pixman-access.obj) : warning LNK4049: 已导入本地定义的符号 free
libpixman-1.lib(pixman-general.obj) : warning LNK4049: 已导入本地定义的符号 free
libpixman-1.lib(pixman-fast-path.obj) : warning LNK4049: 已导入本地定义的符号 free
libpixman-1.lib(pixman-region16.obj) : warning LNK4049: 已导入本地定义的符号 free
libpixman-1.lib(pixman-linear-gradient.obj) : warning LNK4049: 已导入本地定义的符号 free
libpixman-1.lib(pixman-radial-gradient.obj) : warning LNK4217: 本地定义的符号 free 在函数 radial_write_color 中导入
libpixman-1.lib(pixman-region32.obj) : warning LNK4049: 已导入本地定义的符号 free
libpixman-1.lib(pixman-utils.obj) : warning LNK4049: 已导入本地定义的符号 free
libpixman-1.lib(pixman.obj) : warning LNK4217: 本地定义的符号 free 在函数 pixman_image_fill_rectangles 中导入
libpixman-1.lib(pixman-bits-image.obj) : warning LNK4049: 已导入本地定义的符号 free
libpixman-1.lib(pixman-image.obj) : warning LNK4049: 已导入本地定义的符号 free
libpixman-1.lib(pixman-trap.obj) : warning LNK4049: 已导入本地定义的符号 free
libpixman-1.lib(pixman-bits-image.obj) : warning LNK4217: 本地定义的符号 calloc 在函数 _pixman_bits_image_init 中导入
libpixman-1.lib(pixman-utils.obj) : warning LNK4049: 已导入本地定义的符号 malloc
libpixman-1.lib(pixman-fast-path.obj) : warning LNK4049: 已导入本地定义的符号 malloc
libpixman-1.lib(pixman-region16.obj) : warning LNK4217: 本地定义的符号 malloc 在函数 pixman_region_intersect 中导入
libpixman-1.lib(pixman-ssse3.obj) : warning LNK4049: 已导入本地定义的符号 malloc
libpixman-1.lib(pixman-bits-image.obj) : warning LNK4217: 本地定义的符号 malloc 在函数 _pixman_bits_image_init 中导入
libpixman-1.lib(pixman-image.obj) : warning LNK4217: 本地定义的符号 malloc 在函数 _pixman_image_fini 中导入
libpixman-1.lib(pixman-region32.obj) : warning LNK4049: 已导入本地定义的符号 malloc
libpixman-1.lib(pixman-implementation.obj) : warning LNK4049: 已导入本地定义的符号 malloc
libpixman-1.lib(pixman-access.obj) : warning LNK4049: 已导入本地定义的符号 _wassert
libpixman-1.lib(pixman-access-accessors.obj) : warning LNK4049: 已导入本地定义的符号 _wassert
libpixman-1.lib(pixman-bits-image.obj) : warning LNK4217: 本地定义的符号 _wassert 在函数 __bits_image_fetch_affine_no_alpha 中导入
libpixman-1.lib(pixman-image.obj) : warning LNK4049: 已导入本地定义的符号 _wassert
libpixman-1.lib(pixman-matrix.obj) : warning LNK4049: 已导入本地定义的符号 _wassert
libpixman-1.lib(pixman-implementation.obj) : warning LNK4049: 已导入本地定义的符号 _wassert
libpixman-1.lib(pixman-region32.obj) : warning LNK4217: 本地定义的符号 __acrt_iob_func 在函数 pixman_region32_print 中导入
libpixman-1.lib(pixman-implementation.obj) : warning LNK4049: 已导入本地定义的符号 __acrt_iob_func
libpixman-1.lib(pixman-utils.obj) : warning LNK4049: 已导入本地定义的符号 __acrt_iob_func
libpixman-1.lib(pixman-region16.obj) : warning LNK4217: 本地定义的符号 __acrt_iob_func 在函数 pixman_region_init_from_image 中导入
libpixman-1.lib(pixman-region32.obj) : warning LNK4217: 本地定义的符号 __stdio_common_vfprintf 在函数 _vfprintf_l 中导入
libpixman-1.lib(pixman-implementation.obj) : warning LNK4049: 已导入本地定义的符号 __stdio_common_vfprintf
libpixman-1.lib(pixman-utils.obj) : warning LNK4049: 已导入本地定义的符号 __stdio_common_vfprintf
libpixman-1.lib(pixman-region16.obj) : warning LNK4217: 本地定义的符号 __stdio_common_vfprintf 在函数 pixman_region_init_from_image 中导入
libpixman-1.lib(pixman-region32.obj) : warning LNK4217: 本地定义的符号 memmove 在函数 pixman_op 中导入
libpixman-1.lib(pixman-region16.obj) : warning LNK4217: 本地定义的符号 memmove 在函数 _vfprintf_l 中导入
libpixman-1.lib(pixman-sse2.obj) : warning LNK4049: 已导入本地定义的符号 memmove
libpixman-1.lib(pixman-region32.obj) : warning LNK4217: 本地定义的符号 realloc 在函数 pixman_op 中导入
libpixman-1.lib(pixman-region16.obj) : warning LNK4217: 本地定义的符号 realloc 在函数 pixman_coalesce 中导入
libpixman-1.lib(pixman-implementation.obj) : warning LNK4217: 本地定义的符号 getenv 在函数 _pixman_disabled 中导入
libpixman-1.lib(pixman-implementation.obj) : warning LNK4217: 本地定义的符号 strchr 在函数 _pixman_disabled 中导入
libpixman-1.lib(pixman-implementation.obj) : warning LNK4217: 本地定义的符号 strncmp 在函数 _pixman_disabled 中导入
cairo-freed-pool.obj : warning LNK4221: 此对象文件未定义任何之前未定义的公共符号，因此任何耗用此库的链接操作都不会使用此文件
cairo-fixed.obj : warning LNK4221: 此对象文件未定义任何之前未定义的公共符号，因此任何耗用此库的链接操作都不会使用此文件
Built successfully!
You should copy the following files to a proper place now:

        cairo-version.h (NOTE: toplevel, not the src/cairo-version.h one!)
        src/cairo-features.h
        src/cairo.h
        src/cairo-deprecated.h
        src/cairo-win32.h
        src/cairo-script.h
        src/cairo-ps.h
        src/cairo-pdf.h
        src/cairo-svg.h
        src/release/cairo.dll
        src/release/cairo-static.lib
make[1]: Leaving directory '/e/poppler/cairo-1.17.2/src'

E:\cairo\cairo-1.17.2>dumpbin -headers E:\3rd\cairo1.17.2\lib\cairo.dll
Microsoft (R) COFF/PE Dumper Version 14.00.24210.0
Copyright (C) Microsoft Corporation.  All rights reserved.


Dump of file E:\3rd\cairo1.17.2\lib\cairo.dll

PE signature found

File Type: DLL

FILE HEADER VALUES
            8664 machine (x64)
               7 number of sections
        63ABBA66 time date stamp Wed Dec 28 11:39:18 2022
               0 file pointer to symbol table
               0 number of symbols
              F0 size of optional header
            2022 characteristics
                   Executable
                   Application can handle large (>2GB) addresses
                   DLL



E:\cairo\cairo-1.17.2>dumpbin -headers E:\3rd\cairo1.17.2\lib\cairo.lib
Microsoft (R) COFF/PE Dumper Version 14.00.24210.0
Copyright (C) Microsoft Corporation.  All rights reserved.


Dump of file E:\3rd\cairo1.17.2\lib\cairo.lib

File Type: LIBRARY

FILE HEADER VALUES
            8664 machine (x64)
               3 number of sections
        63ABBA65 time date stamp Wed Dec 28 11:39:17 2022
             107 file pointer to symbol table
               8 number of symbols
               0 size of optional header
               0 characteristics

SECTION HEADER #1
.debug$S name
       0 physical address
       0 virtual address
      3F size of raw data
      8C file pointer to raw data (0000008C to 000000CA)
       0 file pointer to relocation table
       0 file pointer to line numbers
       0 number of relocations
       0 number of line numbers
42100040 flags
         Initialized Data
         Discardable
         1 byte align
         Read Only

E:\cairo\cairo-1.17.2>dumpbin -headers E:\3rd\cairo1.17.2\lib\cairo-static.lib
Microsoft (R) COFF/PE Dumper Version 14.00.24210.0
Copyright (C) Microsoft Corporation.  All rights reserved.


Dump of file E:\3rd\cairo1.17.2\lib\cairo-static.lib

File Type: LIBRARY

FILE HEADER VALUES
            8664 machine (x64)
              4D number of sections
        63ABB774 time date stamp Wed Dec 28 11:26:44 2022
            3289 file pointer to symbol table
             10D number of symbols
               0 size of optional header
               0 characteristics

SECTION HEADER #1
.drectve name
       0 physical address
       0 virtual address
      2F size of raw data
     C1C file pointer to raw data (00000C1C to 00000C4A)
       0 file pointer to relocation table
       0 file pointer to line numbers
       0 number of relocations
       0 number of line numbers

```

# 76 . localtime

```
long GetFileVersion(std::string strCfgPath,char *szVersion,long len)
{
	if (_access(strCfgPath.c_str(), 0) != 0)
		return 0;

	if (NULL == szVersion || len < 1)
		return 0;
#ifdef _WIN32
	struct _stat64i32 statbuf;
	struct tm *temptm;
	_stat64i32(strCfgPath.c_str(), &statbuf);
	temptm = _localtime64(&statbuf.st_mtime);
	snprintf(szVersion,len, "%ld%02ld%02ld%02ld%02ld%02ld", (long)(1900 + temptm->tm_year), (long)(1 + temptm->tm_mon), (long)(temptm->tm_mday), (long)(temptm->tm_hour), (long)(temptm->tm_min), (long)(temptm->tm_sec));
#else
	struct _stat     sta;
	int    result = _stat(strCfgPath.c_str(),&sta);
	struct tm *p;
	p = localtime(&sta.st_mtime); //获取当前时间
	if (p != NULL)
	{//格式化时间和随机数字字符串
		int iYear = 1900 + p->tm_year;
		iYear = iYear - (iYear / 100) * 100;
		snprintf(szVersion, len, "%ld%02ld%02ld%02ld%02ld%02ld", (__int64)(iYear), (__int64)(1 + p->tm_mon), (__int64)(p->tm_mday), (__int64)(p->tm_hour), (__int64)(p->tm_min), (__int64)(p->tm_sec));
	}
#endif
	return 1;
}

///// get current time
time_t second;
time(&second);
struct tm *p = localtime(&second); 
if (p != NULL)
{
	int iYear = 1900 + p->tm_year;
	iYear = iYear - (iYear / 100) * 100;
	snprintf(szName, len, "%ld%02ld%02ld%02ld%02ld%02ld", (__int64)(iYear), (__int64)(1 + p->tm_mon), (__int64)(p->tm_mday), (__int64)(p->tm_hour), (__int64)(p->tm_min), (__int64)(p->tm_sec));
}
```

# 77. GetCpuInfo

```
void GetCpuInfo(CString &chProcessorName,CString &chProcessorType,DWORD &dwNum,DWORD &dwMaxClockSpeed)
{
#ifdef _WIN32
	CString strPath=_T("HARDWARE\\DESCRIPTION\\System\\CentralProcessor\\0");//注册表子键路径
	CRegKey regkey;//定义注册表类对象
	LONG lResult;//LONG型变量－反应结果
	lResult=regkey.Open(HKEY_LOCAL_MACHINE,LPCTSTR(strPath),KEY_ALL_ACCESS); //打开注册表键
	if (lResult!=ERROR_SUCCESS)
	{
		return;
	}
	char chCPUName[50] = {0};
	DWORD dwSize=50; 

	//获取ProcessorNameString字段值
	if (ERROR_SUCCESS == regkey.QueryStringValue(_T("ProcessorNameString"),chCPUName,&dwSize))
	{
		chProcessorName = chCPUName;
	}

	//查询CPU主频
	DWORD dwValue;
	if (ERROR_SUCCESS == regkey.QueryDWORDValue(_T("~MHz"),dwValue))
	{
		dwMaxClockSpeed = dwValue;
	}
	regkey.Close();//关闭注册表
	//UpdateData(FALSE);
#else

#endif
	//获取CPU核心数目
	SYSTEM_INFO si;
	memset(&si,0,sizeof(SYSTEM_INFO));
	GetSystemInfo(&si);
	dwNum = si.dwNumberOfProcessors;

	switch (si.dwProcessorType)
	{
	case PROCESSOR_INTEL_386:
		{
			chProcessorType.Format(_T("Intel 386 processor"));
		}
		break;
	case PROCESSOR_INTEL_486:
		{
			chProcessorType.Format(_T("Intel 486 Processor"));
		}
		break;
	case PROCESSOR_INTEL_PENTIUM:
		{
			chProcessorType.Format(_T("Intel Pentium Processor"));
		}
		break;
	case PROCESSOR_INTEL_IA64:
		{
			chProcessorType.Format(_T("Intel IA64 Processor"));
		}
		break;
	case PROCESSOR_AMD_X8664:
		{
			chProcessorType.Format(_T("AMD X8664 Processor"));
		}
		break;
	default:
		chProcessorType.Format(_T("未知"));
		break;
	}

	//GetDisplayName()
}


	__int64 rFuid = 0;
#ifdef _WIN32
	rFuid = _abs64(fuid);
#else
	rFuid = abs(fuid);
#endif
```
# 78  search FileDirPath

```
//==================================API_FileDirPath()===========================
/// @brief 在文件夹下查找文件所在文件夹路径
///
/// @param [in] strDir         查找的文件夹
/// @param [in] strFile        文件名  
///
/// @return 文件夹路径列表（多个只返回第一个）
//================================================================================
MFCOMMONEXPORT long API_FileDirPath(CString strDir, CString strFile, CString& strDirPath)
{
	if (strDir.GetLength() <= 0 || strFile.GetLength() <= 0)
		return 0;

	if (strDir[strDir.GetLength() - 1] != '\\' && strDir[strDir.GetLength() - 1] != '/')
	{
#ifdef _WIN32
		strDir += "\\";
#else
		strDir += "/";
#endif
	}
	string strPath = "";
	//查找文件，返回是否找到

#ifdef _WIN32

	//文件句柄  
	long   hFile = 0;
	//文件信息  
	struct _finddata_t fileinfo;  //很少用的文件信息读取结构
	string p;  //string类很有意思的一个赋值函数:assign()，有很多重载版本
	if ((hFile = _findfirst((strDir + "*").GetBuffer(), &fileinfo)) != -1)
	{
		do
		{
			if ((fileinfo.attrib &  _A_SUBDIR))  //判断是否为文件夹
			{
				if (strcmp(fileinfo.name, ".") != 0 && strcmp(fileinfo.name, "..") != 0)
				{
					CString tempDir = strDir + fileinfo.name;
					if (API_FileDirPath(tempDir, strFile, strDirPath) > 0) 
					{
						_findclose(hFile);
						return 1;
					}
				}
			}
			else    //文件处理
			{
				if (stricmp(strFile.GetBuffer(), fileinfo.name) == 0) 
				{
				
					//获取结果路径
					strPath = strDir.GetBuffer();
					size_t last_slash_idx = strPath.rfind('\\');
					if (std::string::npos != last_slash_idx)
					{
						strDirPath = strPath.substr(0, last_slash_idx).c_str();
					}
					else
					{
						last_slash_idx = strPath.rfind('/');
						if (std::string::npos != last_slash_idx)
						{
							strDirPath = strPath.substr(0, last_slash_idx).c_str();
						}
					}
					_findclose(hFile);
					return 1;
				}
			}
		} while (_findnext(hFile, &fileinfo) == 0);  //寻找下一个，成功返回0，否则-1
	}
	_findclose(hFile);
	return 0;
#else
	DIR *dir;
	struct dirent *ptr;
	long nIndex = 0;

	if ((dir = opendir(strDir.GetBuffer())) == NULL)
	{
		return 0;
	}
	string strFileTmp = strFile.GetBuffer();
	/*nIndex = strFileTmp.find("*");
	if (nIndex != string::npos)
	{
		strFileTmp.erase(nIndex, 1);
	}*/
	while ((ptr = readdir(dir)) != NULL)
	{
		if (strcmp(ptr->d_name, ".") == 0 || strcmp(ptr->d_name, "..") == 0) ///current dir OR parrent dir
			continue;//跳过.和..目录
		else if (ptr->d_type == 8)    ///d_type=8对应file
		{
			string strTmp = ptr->d_name;
			if (stricmp(strTmp.c_str(), strFileTmp.c_str()) == 0)
			{
				strPath = strDir.GetBuffer();
				size_t last_slash_idx = strPath.rfind('\\');
				if (std::string::npos != last_slash_idx)
				{
					strDirPath = strPath.substr(0, last_slash_idx).c_str();
				}
				else
				{
					last_slash_idx = strPath.rfind('/');
					if (std::string::npos != last_slash_idx)
					{
						strDirPath = strPath.substr(0, last_slash_idx).c_str();
					}
				}
				closedir(dir);
				return 1;
			}
		}
		else if (ptr->d_type == 4)    ///d_type=8对应file
		{
			string strTmp = strDir.GetBuffer() + ptr->d_name;
			if (API_FileDirPath(strTmp.c_str(), strFile, strDirPath) > 0) 
			{
				closedir(dir);
				return 1;
			}
		}
	}
	closedir(dir);
	return 0;
#endif
```
```
	//获取系统可用内存大小
	MEMORYSTATUSEX statex;
	DWORDLONG      dwMaxAvailPhys = MAX_CACHE_SIZE;

	statex.dwLength = sizeof(MEMORYSTATUSEX);
#ifdef _WIN32
	GlobalMemoryStatusEx(&statex);
#else
#endif


static size_t GetProcessMemory()
{
#ifdef _WIN32
	HANDLE handle = GetCurrentProcess();
	PROCESS_MEMORY_COUNTERS pmc;
	GetProcessMemoryInfo(handle,&pmc,sizeof(pmc));
	return pmc.WorkingSetSize;
#else
	return 1024;
#endif
}

```

# 79  readelf -h pdfdemo


```
readelf -h pdfdemo
ELF 头：
  Magic：  7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  类别:                              ELF64
  数据:                              2 补码，小端序 (little endian)
  版本:                              1 (current)
  OS/ABI:                            UNIX - System V
  ABI 版本:                          0
  类型:                              EXEC (可执行文件)
  系统架构:                          Advanced Micro Devices X86-64
  版本:                              0x1
  入口点地址：              0x4007b0
  程序头起点：              64 (bytes into file)
  Start of section headers:          37096 (bytes into file)
  标志：             0x0
  本头的大小：       64 (字节)
  程序头大小：       56 (字节)
  Number of program headers:         9
  节头大小：         64 (字节)
  节头数量：         38
  字符串表索引节头： 37


readelf -h mdmapper
ELF 头：
  Magic：  7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  类别:                              ELF64
  数据:                              2 补码，小端序 (little endian)
  版本:                              1 (current)
  OS/ABI:                            UNIX - System V
  ABI 版本:                          0
  类型:                              DYN (共享目标文件)
  系统架构:                          Advanced Micro Devices X86-64
  版本:                              0x1
  入口点地址：              0x960
  程序头起点：              64 (bytes into file)
  Start of section headers:          37064 (bytes into file)
  标志：             0x0
  本头的大小：       64 (字节)
  程序头大小：       56 (字节)
  Number of program headers:         7
  节头大小：         64 (字节)
  节头数量：         36
  字符串表索引节头： 35

```

```
[os@localhost program]$ readelf -h mdmapper
ELF 头：
  Magic：  7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00
  类别:                              ELF64
  数据:                              2 补码，小端序 (little endian)
  版本:                              1 (current)
  OS/ABI:                            UNIX - System V
  ABI 版本:                          0
  类型:                              EXEC (可执行文件)
  系统架构:                          Advanced Micro Devices X86-64
  版本:                              0x1
  入口点地址：              0x400900
  程序头起点：              64 (bytes into file)
  Start of section headers:          61592 (bytes into file)
  标志：             0x0
  本头的大小：       64 (字节)
  程序头大小：       56 (字节)
  Number of program headers:         9
  节头大小：         64 (字节)
  节头数量：         38
  字符串表索引节头： 37

```

# 80  LD_LIBRARY_PATH
export BUTTER_DEV_DIR=/home/os/butter
export BUTTER_DEV_SDK=$BUTTER_DEV_DIR/sdk
export LD_LIBRARY_PATH=$BUTTER_DEV_DIR/bin/program


# 81  locale::facet::_S_create_c_locale name not valid

when running execute file , load the dependent librarys,
some variables in the librarys will be instanced before entry into main function
 
```
Data Load

terminate called after throwing an instance of 'std::runtime_error'
  what():  locale::facet::_S_create_c_locale name not valid

Program received signal SIGABRT, Aborted.

```
From 

```
static const std::locale LOC_CHS("chs"); //简体中文本地化设置
```
 Change To

```
#if (defined(BUTTER_LINUX))                //2017.12.26 cycle
static const std::locale LOC_CHS("zh_CN.UTF-8"); //简体中文本地化设置
#elif defined(BUTTER_ANDROID) || defined(BUTTER_IOS)
static const std::locale LOC_CHS("");
#else
static const std::locale LOC_CHS("chs"); //简体中文本地化设置
#endifd
```

# 82  Delete data in Directory and sub directory

```
//删除dir目录以及其子目录下的所有文件信息
void DeleteDirectoryData(string dir)
{
#ifdef _WIN32
		int nFind = 0;
#else
		int nFind = 1;
#endif
	_finddata_t		fileInfo;
	string			curDir = dir;

	if (curDir[curDir.length() - 1] == '/')
		curDir.append("*");
	else
		curDir.append("/*");

	intptr_t intptr = _findfirst(curDir.c_str(), &fileInfo);//获取目录下的第一个文件信息
	if (intptr == -1) return;
	do
	{
		std::string fileName = fileInfo.name;
		if (fileName == ".." || fileName == ".") continue;
		unsigned int attri = fileInfo.attrib;
		string subdir;
		subdir.append(dir); subdir.append("/"); subdir.append(fileName);

		if ((attri&_A_SUBDIR) == _A_SUBDIR)
		{
			DeleteDirectoryData(subdir);
			rmdir(subdir.c_str());
		}
		else
		{
			//删除文件
			remove(subdir.c_str());
		}
	} while (_findnext(intptr, &fileInfo)  == nFind);//若目录中下一个文件信息不为空

	_findclose(intptr);
}

```

# 83 compile poppler with cmake gui

```
   Unsupported CMAKE_BUILD_TYPE:
   1. set CMAKE_BUILD_TYPE=Release
   2. set FREETYPE_LIBRARY=E:/3rd/freetype/include/freetype
      Set FREETYPE_INCLUDE_DIR_freetype2=E:/3rd/freetype/include/freetype
	  Set FREETYPE_INCLUDE_DIR_ft2build=E:/3rd/freetype/include
   3. JPEG_LIBRARY=E:/3rd/jpeg/lib/libjpeg.lib
      ZLIB_LIBRARY=E:/3rd/zlib/lib/zlib.lib
      PNG_LIBRARY=E:/3rd/png/lib/libpng16.lib
      PNG_PNG_INCLUDE_DIR=E:/3rd/png/include
   
   4. Qt5Core_DIR = E:\3rd\Qt5.12.6\lib\cmake\Qt5Core
      Qt5Gui_DIR  = E:\3rd\Qt5.12.6\lib\cmake\Qt5Gui
	  
   5. OPENJPEG_DIR=E:\3rd\openjpeg-v2.5.0\lib\openjpeg-2.5
   6. GLIB2_LIBRARIES=E:/3rd/glib/glib.lib
      CURL_LIBRARY=E:/3rd/curl/curl.lib
	  
```

# 84. compile libspectre on X64 MSVC


## compile in msys64
```


export LIBPNG_PATH=/e/xspace/3rd/libpng
export ZLIB_PATH=/e/xspace/3rd/zlib
export PIXMAN_PATH=/e/xspace/3rd/pixman


../configure --prefix=/e/cairo/cario --host=x86_64-pc-msys  --enable-shared --enable-static  pixman_CFLAGS='-I/e/xspace/3rd/pixman/include' pixman_LIBS='-L/e/xspace/3rd/pixman/lib -lpixman-1'  png_CFLAGS='-I/e/xspace/3rd/libpng/include' png_LIBS='-L/e/xspace/3rd/libpng/lib  -lpng16' png_REQUIRES='libpng16' CFLAGS='-I/e/xspace/3rd/zlib/include' LIBS='-L/e/xspace/3rd/zlib/lib -lz' --enable-ft=yes  FREETYPE_CFLAGS='-I/e/xspace/3rd/freetype/include' FREETYPE_LIBS='-L/e/xspace/3rd/freetype/lib -lfreetype' --enable-qt=yes  qt_CFLAGS='-I/e/xspace/3rd/Qt5.12.6/include' qt_LIBS='-L/e/xspace/3rd/Qt5.12.6/lib -lQt5Core'



```

## method 1.STEPS on X64 MSVC


```
1. open msvc,add a new empty project named 'libspectre';
2. in project '',append the header files and source files which located in folder 'libspectre-0.2.12\libspectre',
3. maybe we should check the version file 'spectre-version.h',this file based on file 'spectre-version.h.in' can be created by configue in method 2;
4. set the project target dll
5. then we can get the file 'libspectre.lib' and 'libspectre.dll'
```

## method 2.STEPS on X64 MSVC

```
1. get pixman tar file 'libspectre-0.2.11.tar.gz',open the folder in which contain target file;
2. uncompress the tar file above, 
   tar -zxvf  libspectre-0.2.11.tar.gz
3. NOTE: if u compile pixman with the instruction 'end to end build for win32' for cario,
   u will get the x86 static and dynamic library; 
   in order to get the  X64 library,we should use the correct environment;
  
   open visual studio tools 'VS2015 x64 command prompts';
   
4. in vs2015 x64 command prompts window, 
   
   [4.1] firstly,set the environment
   
   set PATH=%PATH%;E:\msys64\usr\bin
   
   [4.2]
   cd $(ROOTDIR)/libspectre
   mkdir build_libspectre
   cd build_libspectre
   
   bash ../configure --prefix=/e/libspectre/ --host=x86_64-pc-msys  --enable-shared --enable-static  CFLAGS='-I/e/xspace/3rd/gs10/include' LIBS='-L/e/xspace/3rd/gs10.00.0/bin  -lgs' CAIRO_CFLAGS='-I/e/xspace/3rd/cairo1.17.2/include' CAIRO_LIBS='-L/e/xspace/3rd/cairo1.17.2/lib  -lcairo' 
   
   
   make -j4

5. then in folder $(DIR)\libspectre-0.2.11/src/release/ , we can get the target lib 'cairo.dll';
   
6. check the library with dumpbin command;
   dumpbin -headers $(DIR)\libspectre-0.2.11\lib\cairo.dll
   

```
# 85. import osm data into postgreSQL with osm2pgsql


## STEPS

```

1.put the postgresql bin dir into PATH;
  then we can find createdb command in cmd dialog;

2.create datebase in command dialog
  createdb -U postgres osm_china

  or open 'pgAdmin' create database

3.create extension

3.0 with 'pgAdmin'
    in pgAdmin, select database, right click 'Query Tool', then open query editor ,in editor ,input 
	CREATE EXTENSION postgis;
	then run it;

3.1 with Navicat link postgreSQL and load database osm_china,new search command,in command dlg,put the Following string
   CREATE EXTENSION postgis; 
   the RUN it；in the information dialog,can see 'ok';
   
3.2 with command
  psql -U postgres -d osm_wh -f E:\postgresql-12.1-3\pgsql\share\contrib\postgis-3.0\postgis.sql
  psql -U postgres -d osm_wh -f E:\postgresql-12.1-3\pgsql\share\contrib\postgis-3.0\spatial_ref_sys.sql
	
4.0 import osm datas; 

osm2pgsql -d osm_china -U postgres -P 5432 -C 12000 -S "F:/osm2pgsql-bin/default.style" F:/osmdata/wh-latest.osm.pbf
 ```

## NOTE:

  method 3.2 DO NOT create postgis EXtension well;perhaps need other commands besides the two above;
  method 3.1 DO Create postgis extension well;
  method 3.0 DO Create postgis extension well;method 3.0 is recommended;

  Because when import osm data with method 3.2,we crush error:
  
  ```
	F:\osm2pgsql-bin>osm2pgsql -d osm_wh -U postgres -P 5432 -C 12000 -S "F:/osm2pgsql-bin/default.style" F:/osmdata/wh-latest.osm.pbf
    2023-02-28 15:39:18  osm2pgsql version 1.8.1
    2023-02-28 15:39:18  ERROR: The postgis extension is not enabled on the database 'osm_wh'. Are you using the correct database? Enable with 'CREATE EXTENSION postgis;'
 ```

when import osm data with method 3.1,we import well:

osm2pgsql -d osm_china -U postgres -P 5432 -C 12000 -S "F:/osm2pgsql-bin/default.style" F:/osmdata/wh-latest.osm.pbf

```
F:\osm2pgsql-bin>osm2pgsql -d osm_china -U postgres -P 5432 -C 12000 -S "F:/osm2pgsql-bin/default.style" F:/osmdata/wh-latest.osm.pbf
2023-02-28 15:40:41  osm2pgsql version 1.8.1
2023-02-28 15:40:41  Database version: 12.1
2023-02-28 15:40:41  PostGIS version: 3.0
2023-02-28 15:40:42  Setting up table 'planet_osm_point'
2023-02-28 15:40:42  Setting up table 'planet_osm_line'
2023-02-28 15:40:42  Setting up table 'planet_osm_polygon'
2023-02-28 15:40:42  Setting up table 'planet_osm_roads'

2023-02-28 15:40:46  Reading input files done in 3s.
2023-02-28 15:40:46    Processed 1631766 nodes in 0s - 1632k/s
2023-02-28 15:40:46    Processed 139513 ways in 1s - 140k/s
2023-02-28 15:40:46    Processed 461 relations in 2s - 230/s
2023-02-28 15:40:48  Clustering table 'planet_osm_line' by geometry...
2023-02-28 15:40:48  Clustering table 'planet_osm_polygon' by geometry...
2023-02-28 15:40:48  Clustering table 'planet_osm_point' by geometry...
2023-02-28 15:40:48  Clustering table 'planet_osm_roads' by geometry...
2023-02-28 15:40:49  Creating geometry index on table 'planet_osm_point'...
2023-02-28 15:40:49  Creating geometry index on table 'planet_osm_roads'...
2023-02-28 15:40:49  Analyzing table 'planet_osm_roads'...
2023-02-28 15:40:49  Analyzing table 'planet_osm_point'...
2023-02-28 15:40:49  All postprocessing on table 'planet_osm_point' done in 1s.
2023-02-28 15:40:50  Creating geometry index on table 'planet_osm_line'...
2023-02-28 15:40:50  Analyzing table 'planet_osm_line'...
2023-02-28 15:40:50  All postprocessing on table 'planet_osm_line' done in 2s.
2023-02-28 15:40:51  Creating geometry index on table 'planet_osm_polygon'...
2023-02-28 15:40:52  Analyzing table 'planet_osm_polygon'...
2023-02-28 15:40:52  All postprocessing on table 'planet_osm_polygon' done in 4s
.
2023-02-28 15:40:52  All postprocessing on table 'planet_osm_roads' done in 1s.
2023-02-28 15:40:52  osm2pgsql took 11s overall.
```

..\osm2pgsql.exe -U postgres -G --hstore --style openstreetmap-carto.style --tag-transform-script openstreetmap-carto.lua -d gis F:\mapov.osm


## createdb options and osm2pgsql options
```
F:\osm2pgsql-bin>createdb --help
createdb 创建一个 PostgreSQL 数据库.

使用方法:
  createdb [选项]... [数据库名称] [描述]

选项:
  -D, --tablespace=TABLESPACE  数据库默认表空间
  -e, --echo                   显示发送到服务端的命令
  -E, --encoding=ENCODING      数据库编码
  -l, --locale=LOCALE          数据库的本地化设置
      --lc-collate=LOCALE      数据库的LC_COLLATE设置
      --lc-ctype=LOCALE        数据库的LC_CTYPE设置
  -O, --owner=OWNER            新数据库的所属用户
  -T, --template=TEMPLATE      要拷贝的数据库模板
  -V, --version                输出版本信息, 然后退出
  -?, --help                   显示此帮助, 然后退出

联接选项:
  -h, --host=HOSTNAME          数据库服务器所在机器的主机名或套接字目录
  -p, --port=PORT              数据库服务器端口号
  -U, --username=USERNAME      联接的用户名
  -w, --no-password            永远不提示输入口令
  -W, --password               强制提示输入口令
  --maintenance-db=DBNAME      更改维护数据库

默认情况下, 以当前用户的用户名创建数据库.

臭虫报告至 <pgsql-bugs@lists.postgresql.org>.

F:\osm2pgsql-bin>osm2pgsql --help
2023-02-28 15:51:50  osm2pgsql version 1.8.1

Usage: osm2pgsql [OPTIONS] OSM-FILE...

Import data from the OSM file(s) into a PostgreSQL database.

Full documentation is available at https://osm2pgsql.org/

Common options:
    -a|--append     Update existing osm2pgsql database with data from file.
    -c|--create     Import OSM data from file into database. This is the
                    default if --append is not specified.
    -O|--output=OUTPUT  Set output. Options are:
                    pgsql - Output to a PostGIS database (default)
                    flex - More flexible output to PostGIS database
                    gazetteer - Output to a PostGIS database for Nominatim
                                (deprecated)
                    null - No output. Used for testing.
    -S|--style=FILE  Location of the style file. Defaults to
                    'default.style'.
    -k|--hstore     Add tags without column to an additional hstore column.
       --tag-transform-script=SCRIPT  Specify a Lua script to handle tag
                    filtering and normalisation (pgsql output only).
    -s|--slim       Store temporary data in the database. This switch is
                    required if you want to update with --append later.
        --drop      Only with --slim: drop temporary tables after import
                    (no updates are possible).
    -C|--cache=SIZE  Use up to SIZE MB for caching nodes (default: 800).
    -F|--flat-nodes=FILE  Specifies the file to use to persistently store node
                    information in slim mode instead of in PostgreSQL.
                    This is a single large file (> 50GB). Only recommended
                    for full planet imports. Default is disabled.

Database options:
    -d|--database=DB  The name of the PostgreSQL database to connect to or
                    a PostgreSQL conninfo string.
    -U|--username=NAME  PostgreSQL user name.
    -W|--password   Force password prompt.
    -H|--host=HOST  Database server host name or socket location.
    -P|--port=PORT  Database server port.

Run 'osm2pgsql --help --verbose' (-h -v) for a full list of options.

```

# 86. exclude Seg 

```

 /*   orgSeg                   |------------------|
  *   excludeSeg    |----------------------------------------|   case 1
  *   excludeSeg    |------|                                     case 2
  *   excludeSeg                                      |------|   case 2
  *   excludeSeg    |-----------------|                          case 3
  *   excludeSeg                           |-----------------|   case 4
  *   excludeSeg                  |-----------|                  case 5
  */

struct segment
{
	bool operator < (const segment& seg)
	{
		return min < seg.max;
	}
	int min;
	int max;
};

/// @brief 计算从原始区间段中排除指定区间段后的结果
///
/// @param [in] orgSegs     original segments
/// @param [in] excludeSegs some segments which will be excluded
///
/// @return the segments after exclude excludeSegs

std::vector<segment> ExcludeSegs( const std::vector<segment>& orgSegs,const std::vector<segment>& excludeSegs )
{
	std::vector<segment> org = orgSegs;
	std::vector<segment> segsRtn;
	segment         seg;
	for (int jj = 0; jj < excludeSegs.size(); ++jj)
	{
		segsRtn.clear();
		const segment& excludeSeg = excludeSegs[jj];
	
		for (int ii = 0; ii < org.size(); ++ii)
		{
			const segment& orgSeg = org[ii];

            /// case 1 , exlude totally
			if (orgSeg.min >= excludeSeg.min && orgSeg.max <= excludeSeg.max)
			{
				continue;
			}
            /// case 2 , no crash
			if (orgSeg.min >= excludeSeg.max || excludeSeg.min >= orgSeg.max)
			{
				segsRtn.push_back(orgSeg);
				continue;
			}
            /// case 3 , clip left segment
			if (orgSeg.min >= excludeSeg.min && orgSeg.max >= excludeSeg.max)
			{
				seg.min = excludeSeg.max;
				seg.max = orgSeg.max;
				segsRtn.push_back(seg);
				continue;
			}
            /// case 4 , clip right segment
			if (orgSeg.min <= excludeSeg.min && orgSeg.max <= excludeSeg.max)
			{
				seg.min = orgSeg.min;
				seg.max = excludeSeg.min;
				segsRtn.push_back(seg);
				continue;
			}
            /// case 5 , clip middle segment
			if (orgSeg.min <= excludeSeg.min && orgSeg.max >= excludeSeg.max)
			{
				seg.min = orgSeg.min;
				seg.max = excludeSeg.min;
				segsRtn.push_back(seg);

				seg.min = excludeSeg.max;
				seg.max = orgSeg.max;
				segsRtn.push_back(seg);
				continue;
			}
		}
		org = segsRtn;
	}
	return segsRtn;
}

```


# 87. configure postgreSQL 

```
STEPS：

1. download zip file "postgresql-13.10-1-windows-x64-binaries.zip“
   Unzip it;
  首先把文件夹移动到准备安装的位置，我这里移动到了D:\PostgreSQL 路径

  启动 cmd 进入我们的路径D:\PostgreSQL 切入到 bin 文件夹中

	cmd
	d:
	cd D:\PostgreSQL
	cd bin

2.
	首先初始化实例

	initdb -D "D:\PostgreSQL\data" -E UTF8 -U postgres --locale="Chinese (Simplified)_China.936" --lc-messages="Chinese_China.936" -A scram-sha-256 -W

	在 windows 环境下我们采用 UTF8 编码Chinese (Simplified)_China.936 排序规则，账户加密方式采用scram-sha-256，
	数据库的存放位置指定为D:\PostgreSQL\data



	初始化过程中需要输入两次 超级用户口令，用于设置 postgres 用户的密码

	数据库初始化完成之后，就可以选择安装为 Windows 服务了，
	
3. 注册服务命令如下

	pg_ctl.exe register -D "D:\PostgreSQL\data" -PostgreSQL


4.
	接下来我们调整一下 PostgreSQL 的配置信息，默认情况下 PostgreSQL 数据库只能本机连接，
	我们调整为监听所有 IP 开启外部连接的功能。

	在D:\PostgreSQL\data 文件夹中找到 postgresql.conf

	打开postgresql.conf 文件，找到

	#listen_addresses = 'localhost'

	然后删除掉前面的 # 修改为

	listen_addresses = '*'

	保存后关闭文件。


 5.
	然后还是在D:\PostgreSQL\data 文件夹中找到pg_hba.conf 打开后直接情况里面原来的内容，用如下内容进行替换

	host all all 0.0.0.0/0 scram-sha-256
	host all all ::/0 scram-sha-256

	保存后关闭即可，这样就运行了所有的 ipv4 和 ipv6 地址来连接我们的 PostgreSQL 数据库了，
6.
	当配置文件调整之后我们就可以启动我们安装好的 PostgreSQL 了，
	只要在 cmd 输入

	net start PostgreSQL

	也可以通过 Windows 服务来控制启动和停止

```

# 88. system PATH and User PATH 

## system PATH has higher priority than user PATH

when find some dynamic library,search in system PATH firstly,if find the target,load it;
if not find,search the user PATH;
when U has some target library which have different version,maybe load the wrong library;

```

```
#89 QT_PLUGIN_PATH


excutable program 'docker_qt_text' depends on QtCore and QtGUI;
when run it in linux environment, crash the following problem;

```
./program/docker_qt_text in textindocker_InitInstance
qt.qpa.plugin: Could not find the Qt platform plugin "offscreen" in ""
This application failed to start because no Qt platform plugin could be initialized. Reinstalling the application may fix this problem.

已放弃

```
the solution is :

```
export QT_PLUGIN_PATH=/opt/apps/server-task/program/QtPlugins

```

# 90 how to install font in linux



更新字体缓存，使用如下命令：
cd /usr/share/fonts/
mkfontscale
mkfontdir
fc-cache

```
[root@scm fonts]# cd /usr/share/fonts/
[root@scm fonts]# mkfontscale
[root@scm fonts]# mkfontdir
[root@scm fonts]# fc-cache -fv 
/usr/share/fonts: caching, new cache contents: 2 fonts, 0 dirs
/usr/share/X11/fonts/Type1: skipping, no such directory
/usr/share/X11/fonts/TTF: skipping, no such directory
/usr/local/share/fonts: skipping, no such directory
/root/.fonts: skipping, no such directory
/var/cache/fontconfig: cleaning cache directory
/root/.fontconfig: not cleaning non-existent cache directory
fc-cache: succeeded


[root@6a906d8c6750 ]# cd /usr/share/fonts/
[root@6a906d8c6750 fonts]# mkfontscale
[root@6a906d8c6750 fonts]# mkfontdir
[root@6a906d8c6750 fonts]# mkfontscale
[root@6a906d8c6750 fonts]# fc-cache -fv
Font directories:
        /usr/share/fonts
        /usr/share/X11/fonts/Type1
        /usr/share/X11/fonts/TTF
        /usr/local/share/fonts
        /root/.local/share/fonts
        /root/.fonts
        /usr/share/fonts/cantarell
        /usr/share/fonts/chinese
        /usr/share/fonts/chinese/noto
/usr/share/fonts: caching, new cache contents: 0 fonts, 2 dirs
/usr/share/fonts/cantarell: caching, new cache contents: 5 fonts, 0 dirs
/usr/share/fonts/chinese: caching, new cache contents: 76 fonts, 1 dirs
/usr/share/fonts/chinese/noto: caching, new cache contents: 73 fonts, 0 dirs
/usr/share/X11/fonts/Type1: skipping, no such directory
/usr/share/X11/fonts/TTF: skipping, no such directory
/usr/local/share/fonts: skipping, no such directory
/root/.local/share/fonts: skipping, no such directory
/root/.fonts: skipping, no such directory
/usr/share/fonts/cantarell: skipping, looped directory detected
/usr/share/fonts/chinese: skipping, looped directory detected
/usr/share/fonts/chinese/noto: skipping, looped directory detected
/usr/lib/fontconfig/cache: cleaning cache directory
/root/.cache/fontconfig: not cleaning non-existent cache directory
/root/.fontconfig: not cleaning non-existent cache directory

```

# 91 remove or link file

[root@6a906d8c6750 program]# rm -f ./libQt5Gui.so.5
[root@6a906d8c6750 program]# rm -f ./libQt5Gui.so
[root@6a906d8c6750 program]# ln -s ./libQt5Core.so.5.12.11 ./libQt5Core.so.5
[root@6a906d8c6750 program]# ln -s ./libQt5Core.so.5.12.11 ./libQt5Core.so


# 92 copy file from or to docker

```
在宿主机里面执行以下命令：

docker cp 容器名：要拷贝的文件在容器里面的路径       要拷贝到宿主机的相应路径

示例：

假设容器名为testtomcat,要从容器里面拷贝的文件路为：/usr/local/tomcat/webapps/test/js/test.js,
 现在要将test.js从容器里面拷到宿主机的/opt路径下面，那么命令应该怎么写呢？

在宿主机上面执行命令：
docker cp testtomcat：/usr/local/tomcat/webapps/test/js/test.js /opt

docker cp xxxx:/etc/mysql/my.cnf /home/tom/
注意这个xxxx是docker ps -a 获取的container id

docker cp test:/server-task/program/Config/font113053057.log /home/font/log
docker cp /opt/apps/Qt5/libQt5Core.so  test:/server-task/program
docker cp /opt/apps/Qt5/libQt5Core.so.5  test:/server-task/program
docker cp /opt/apps/Qt5/libQt5Core.so.5.12  test:/server-task/program
```

 # 93 将main1.c重命名为main.c

 [root@6a906d8c6750 program]# rename main1.c main.c main1.c

# 94 REMOTE HOST IDENTIFICATION HAS CHANGED

linux 远程连接ssh提示IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY解决
 
 ```
MacBook-Air:BurpLoader4burpsuite_pro_v1.5.11 watsy$ ssh root@192.168.2.108
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
07:36:8e:d0:12:78:38:f7:21:10:c1:12:d6:31:ad:55.
Please contact your system administrator.
Add correct host key in /Users/watsy/.ssh/known_hosts to get rid of this message.
Offending RSA key in /Users/watsy/.ssh/known_hosts:1
RSA host key for 192.168.2.108 has changed and you have requested strict checking.
Host key verification failed.
 

SOLUTION:
remove or delete file 'known_hosts'
rm -rf ~/.ssh/known_hosts
```
# 95 logger class

```

class Logger
{
public:
	Logger() {
		InitLogPath();
	}
	~Logger() {

	}

	short Info(const char* szTag, const char* strFormat, ...)
	{
		std::string strInfo;
		if (strFormat)
		{
			va_list valist;
			va_start(valist, strFormat);
			strInfo.FormatV(strFormat, valist);
			va_end(valist);
		}
		// 得到时间
		SYSTEMTIME	sysTime = { 0 };
		GetLocalTime(&sysTime);
		char szVal[256] = "\0";
		sprintf(szVal,"%02d:%02d:%02d.%03d | %s | %s", sysTime.wHour, sysTime.wMinute, sysTime.wSecond, sysTime.wMilliseconds, szTag, strInfo.c_str());

		std::string strMsg(szVal);
		return Flush(strMsg.c_str(), strMsg.GetLength());
	}

private:

	std::string  m_szLogPath;

protected:

	long Flush(const char* pByte, long long len)
	{
		return Save(pByte, len, m_szLogPath.c_str());
	}

	std::string GetCurrentTimeString()
	{
		// 得到时间
		SYSTEMTIME	sysTime = { 0 };
		GetLocalTime(&sysTime);
		char szVal[256] = "\0";
		sprintf(szVal,"%02d%02d%02d%03d", sysTime.wHour, sysTime.wMinute, sysTime.wSecond, sysTime.wMilliseconds);

		std::string strTime(szVal);
		return strTime;
	}

	long Save(const char* pByte, long long len, const char* lpFileName)
	{
		if (!lpFileName)
		{
			return 0;
		}
		FILE* pFile = fopen(lpFileName, "rb+");
		if (!pFile) {
			pFile = fopen(lpFileName, "wb+");//create file if not exist
		}
		else {
			fclose(pFile);
			pFile = fopen(lpFileName, "ab+");
		}

		if (!pFile) { return 0; }

		fwrite(pByte, len * sizeof(byte), 1, pFile);
		fclose(pFile);
		return 1;
	}

	void ReplaceSubString(string& str, const char* lpszOld, const char* lpszNew)
	{
		int index = str.find(lpszOld);

		while (index > -1)
		{
			str.replace(index, strlen(lpszOld), lpszNew);
			index = str.find(lpszOld);
		}
	}

	std::string InitLogPath()
	{
		std::string szPath = GetConfigDir(CFG_DIR_ROOT);
		char szVal[256] = "\0";
		sprintf(szVal,"%s\\font%s.log", szPath.c_str(), GetCurrentTimeString().c_str());
		szPath = szVal;
		std::string strPath(szPath.c_str());
		#ifndef _WIN32
		ReplaceSubString(strPath, "\\", "/");
		#endif
		m_szLogPath = strPath.c_str();
		g_Log("InitFont", "LogPath: %s", strPath.c_str());
		return szPath;
	}

};

```

# 96 convert const char* to QString

```
#include <memory>
std::wstring str2wstr(const std::string& str) {
	if (str.empty()) {
		return L"";
	}
	unsigned len = str.size() + 1;
	setlocale(LC_CTYPE, "zh_CN.GBK"); //zh_CN.UTF-8
	std::unique_ptr<wchar_t[]> p(new wchar_t[len]);
	mbstowcs(p.get(), str.c_str(), len);
	std::wstring w_str(p.get());
	return w_str;
}

short g_Log(const char* szTag, const char* strFormat, ...)
{
	return 1;
}

QString makeQStringLiteral(const wchar_t * _wstring, size_t len)
{
	// how to convert const char* to char16_t*;
	const int nSize = 1024;
	QStaticStringData<nSize> qstring_literal;

	const char* pszTagw = "Literal";

	if (sizeof(wchar_t) == sizeof(char16_t))
	{
		const qunicodechar * pszBuffer = (char16_t *)_wstring;
		memcpy(qstring_literal.data, pszBuffer, len * sizeof(qunicodechar));
	}
	else
	{
		for (size_t m = 0; m < len; m++)
		{
			wchar_t val = _wstring[m];
			g_Log(pszTagw, "Z:%d/%d : %d|-> %d \r\n", len, m, _wstring[m], val);
			qstring_literal.data[m] = val;
		}
	}


	for (size_t m = 0; m < len; m++)
	{
		g_Log(pszTagw, "I:%d/%d : %d \r\n", len, m, _wstring[m]);
		g_Log(pszTagw, "O:%d/%d : %d \r\n", len, m, qstring_literal.data[m]);
	}

	qstring_literal.str.ref.atomic = -1;
	qstring_literal.str.size = len;
	qstring_literal.str.alloc = 0;
	qstring_literal.str.capacityReserved = 0;
	qstring_literal.str.offset = sizeof(QTypedArrayData<ushort>);

	QStringDataPtr holder = { qstring_literal.data_ptr() };
	const QString qstring_literal_temp(holder);
	return qstring_literal_temp;

}


QString Cvt2QString(const char* pszText)
{
	if (!pszText) return "";
	std::string strText(pszText);
	std::wstring wstr = str2wstr(strText);

	const char16_t * _wstring16 = (char16_t*)wstr.data();
	const wchar_t * _wstring = (wchar_t*)wstr.data();
	size_t len = wstr.size();
	for (size_t m = 0; m < len; m++)
	{
		g_Log("Cvt", "I:%d/%d : %d \r\n", len, m, _wstring[m]);
	}
	return makeQStringLiteral(_wstring, len);
}

```

# 97  get current time string

```
std::string  getCurTime(bool bFmt = true)
{
	char szTime[MAX_PATH] = { '\0' };
#ifdef _WIN32
	// 得到时间
	SYSTEMTIME	sysTime = { 0 };
	GetLocalTime(&sysTime);
	if(bFmt)
		sprintf_s(szTime, "%02d%02d %02d:%02d:%02d.%03d",sysTime.wMonth,sysTime.wDay, sysTime.wHour, sysTime.wMinute, sysTime.wSecond, sysTime.wMilliseconds);
	else
		sprintf_s(szTime, "%02d%02d%02d%02d%02d%03d", sysTime.wMonth, sysTime.wDay, sysTime.wHour, sysTime.wMinute, sysTime.wSecond, sysTime.wMilliseconds);
#else
	time_t timep;
	struct tm *p;
	time(&timep);
	p = localtime(&timep);
	if(bFmt)
		sprintf(szTime, "%02d.%02d %02d:%02d:%02d.%02d",p->tm_mon + 1,p->tm_mday, p->tm_hour, p->tm_min, p->tm_sec, p->tm_sec);
	else
		sprintf(szTime, "%02d%02d%02d%02d%02d%02d", p->tm_mon + 1, p->tm_mday, p->tm_hour, p->tm_min, p->tm_sec, p->tm_sec);
#endif
	std::string strTime = szTime;
	return strTime;
}
```

# 98 configure with CFLAGS and CC

./configure --prefix=/home/code/libiconv  CC="g++" CFLAGS="-fsigned-char -fdiagnostics-color -std=c++11 -fstack-protector-all -g -O0 -fPIE  -MD    -std=c++11"

```

`configure' configures libiconv 1.16 to adapt to many kinds of systems.

Usage: ./configure [OPTION]... [VAR=VALUE]...

To assign environment variables (e.g., CC, CFLAGS...), specify them as
VAR=VALUE.  See below for descriptions of some of the useful variables.

Defaults for the options are specified in brackets.

Configuration:
  -h, --help              display this help and exit
      --help=short        display options specific to this package
      --help=recursive    display the short help of all the included packages
  -V, --version           display version information and exit
  -q, --quiet, --silent   do not print `checking ...' messages
      --cache-file=FILE   cache test results in FILE [disabled]
  -C, --config-cache      alias for `--cache-file=config.cache'
  -n, --no-create         do not create output files
      --srcdir=DIR        find the sources in DIR [configure dir or `..']

Installation directories:
  --prefix=PREFIX         install architecture-independent files in PREFIX
                          [/usr/local]
  --exec-prefix=EPREFIX   install architecture-dependent files in EPREFIX
                          [PREFIX]

By default, `make install' will install all the files in
`/usr/local/bin', `/usr/local/lib' etc.  You can specify
an installation prefix other than `/usr/local' using `--prefix',
for instance `--prefix=$HOME'.

For better control, use the options below.

Fine tuning of the installation directories:
  --bindir=DIR            user executables [EPREFIX/bin]
  --sbindir=DIR           system admin executables [EPREFIX/sbin]
  --libexecdir=DIR        program executables [EPREFIX/libexec]
  --sysconfdir=DIR        read-only single-machine data [PREFIX/etc]
  --sharedstatedir=DIR    modifiable architecture-independent data [PREFIX/com]
  --localstatedir=DIR     modifiable single-machine data [PREFIX/var]
  --libdir=DIR            object code libraries [EPREFIX/lib]
  --includedir=DIR        C header files [PREFIX/include]
  --oldincludedir=DIR     C header files for non-gcc [/usr/include]
  --datarootdir=DIR       read-only arch.-independent data root [PREFIX/share]
  --datadir=DIR           read-only architecture-independent data [DATAROOTDIR]
  --infodir=DIR           info documentation [DATAROOTDIR/info]
  --localedir=DIR         locale-dependent data [DATAROOTDIR/locale]
  --mandir=DIR            man documentation [DATAROOTDIR/man]
  --docdir=DIR            documentation root [DATAROOTDIR/doc/libiconv]
  --htmldir=DIR           html documentation [DOCDIR]
  --dvidir=DIR            dvi documentation [DOCDIR]
  --pdfdir=DIR            pdf documentation [DOCDIR]
  --psdir=DIR             ps documentation [DOCDIR]

Program names:
  --program-prefix=PREFIX            prepend PREFIX to installed program names
  --program-suffix=SUFFIX            append SUFFIX to installed program names
  --program-transform-name=PROGRAM   run sed PROGRAM on installed program names

System types:
  --build=BUILD     configure for building on BUILD [guessed]
  --host=HOST       cross-compile to build programs to run on HOST [BUILD]

Some influential environment variables:
  CC          C compiler command
  CFLAGS      C compiler flags
  LDFLAGS     linker flags, e.g. -L<lib dir> if you have libraries in a
              nonstandard directory <lib dir>
  LIBS        libraries to pass to the linker, e.g. -l<library>
  CPPFLAGS    (Objective) C/C++ preprocessor flags, e.g. -I<include dir> if
              you have headers in a nonstandard directory <include dir>
  CPP         C preprocessor
  LT_SYS_LIBRARY_PATH
              User-defined run-time library search path.

```

# 99 find and locate
```
//在根目录下查找文件名为mysql的文件夹
[root@localhost ~]# find / -name mysql　　　　

//目录“/usr/local/mysql”中搜索以.bin结尾的所有文件
[root@localhost ~]# find /user/local/mysql -name \*.bin　　

[root@localhost /]# locate \*.log　　　　//查找后缀为.log的文件

[root@localhost /]# locate /etc/my　　// 搜索etc目录下所有以my开头的文件
```


# 100 where the third library 'libiconv.so' Go?

in linux,the execute file 'dockertext' depends on the third party 'libiconv.so';
for make file like bellow;

```
#user  makefile
#define user develop env; eg: include dir, output dir, link dir;
CurDir := $(shell pwd)
ICONV_INC :=$(CurDir)/iconv/include
#define user project information;
EXECUTABLE := $(LIBPATH)/dockertext
LIBS := iconv  
CC:= g++
Z_DEBUG := -g -O0
CFLAGS :=  -I$(CurDir) -I$(ICONV_INC) $(Z_DEBUG)

IS_SO := $(findstring .so,$(EXECUTABLE))
ifneq ($(IS_SO),)
    CFLAGS += $(SO_CFLAGS)
    LDFLAGS += $(SO_LDFLAGS)
else
    CFLAGS += $(EXE_CFLAGS)
    LDFLAGS += $(EXE_LDFLAGS)
endif
CXXFLAGS := $(CFLAGS) 
CPPFLAGS += -MD    -std=c++11

RM-F := rm -f 
RM := rm

SOURCE := $(wildcard *.c) $(wildcard *.cpp) $(foreach n, $(SubDir), $(wildcard $(CurDir)/$(n)/*.cpp)) $(foreach n, $(SubDir), $(wildcard $(CurDir)/$(n)/*.c))
OBJS := $(patsubst %.c,%.o,$(patsubst %.cpp,%.o,$(SOURCE))) 
DEPS := $(patsubst %.o,%.d,$(OBJS)) 
MISSING_DEPS := $(filter-out $(wildcard $(DEPS)),$(DEPS)) 
MISSING_DEPS_SOURCES := $(wildcard $(patsubst %.d,%.cpp,$(MISSING_DEPS)) $(patsubst %.d,%.cpp,$(MISSING_DEPS))) 

..PHONY : build  deps objs clean allclean rebuild 
build: $(EXECUTABLE) 
deps : $(DEPS) 
objs : $(OBJS) 
test :
	echo $(SOURCE)
clean :  
	$(foreach n, $(OBJS), $(shell $(RM) $(n)))
	$(foreach n, $(DEPS), $(shell $(RM) $(n)))
allclean: clean 
	$(RM-F) $(EXECUTABLE) 
rebuild: allclean build
prepare:

header:

ifneq ($(MISSING_DEPS),) 
$(MISSING_DEPS): 
	$(RM-F) $(patsubst %.d,%.o,$@) 
endif 
-include $(DEPS) 
$(EXECUTABLE) : $(OBJS) 
	$(CC) -o $(EXECUTABLE) $(OBJS)  -L$(LIBPATH) $(addprefix -l,$(LIBS)) $(LDFLAGS)
	ld $(EXECUTABLE) -rpath $(LIBPATH)

```

in file dockertext.cpp,we have following logic

```
#include "iconv.h"
#include <string.h>
void textindocker::Run()
{
	const char* out_encode = "UTF-8"; const char* in_encode = "GB18030";
	iconv_t h = iconv_open(out_encode, in_encode);
	if (h == (iconv_t)(-1))
	{
		printf("\r\n Error:GB18030 iconv open fail\r\n");
		return;
	}
	char* psIn = "中"; size_t mm = strlen(psIn);
	size_t n = 0; char* psOut = new char;
	iconv(h, &psIn, &mm, &psOut, &n);
	iconv_close(h);
	delete psOut;
}

```
we compile the third party libiconv and target exec file with debug mode;
after we compile ,we get the target file  dockertext; when debug execute file dockertext,
we cannot debug  the third party library libiconv.so;
HERE is the QUESTION.
in execute file dockertext,we actually use the interface function such as 'iconv_open' 'iconv'
 'iconv_close' which is in library libiconv.so,but why cannot debug it even if we have the debug information?


 1. ok, let me see the depend library files for target execute 'dockertext'

```
root@zone:/opt/apps/zone/program# ldd ./dockertext
    linux-vdso.so.1 (0x0000002023068000)
    libstdc++.so.6 => /lib/aarch64-linux-gnu/libstdc++.so.6 (0x00000020230c1000)
    libgcc_s.so.1 => /lib/aarch64-linux-gnu/libgcc_s.so.1 (0x00000020232a6000)
    libc.so.6 => /lib/aarch64-linux-gnu/libc.so.6 (0x00000020232ca000)
    /lib/ld-linux-aarch64.so.1 (0x0000002023046000)
    libm.so.6 => /lib/aarch64-linux-gnu/libm.so.6 (0x000000202343d000)
```
 from above information,we cannot find the library 'libiconv.so' or related library.
 but we actually use the interface function such as 'iconv_open' 'iconv' 'iconv_close' 
 which is in library libiconv.so, how can it happen?

or the interface function can e found in dependent libarary other than 'libiconv.so';

2. NEXT search the iconv-related funntions in execte dockertext;

```
root@zone:/opt/apps/zone/program# readelf -a ./dockertext | grep iconv
000000014e08  000500000402 R_AARCH64_JUMP_SL 0000000000000000 iconv_close@GLIBC_2.17 + 0
000000014e48  000e00000402 R_AARCH64_JUMP_SL 0000000000000000 iconv_open@GLIBC_2.17 + 0
000000014ec8  001e00000402 R_AARCH64_JUMP_SL 0000000000000000 iconv@GLIBC_2.17 + 0
     5: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND iconv_close@GLIBC_2.17 (4)
    14: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND iconv_open@GLIBC_2.17 (4)
    30: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND iconv@GLIBC_2.17 (4)
   145: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND iconv_close@@GLIBC_2.17
   183: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND iconv_open@@GLIBC_2.17
   230: 0000000000000000     0 FUNC    GLOBAL DEFAULT  UND iconv@@GLIBC_2.17
```
here we found iconv subfix '@GLIBC_2.17' which mean use the iconv-related function in 'glibc' library;
this is the important key information;
from the dependcy above ,we find this line 
```
libc.so.6 => /lib/aarch64-linux-gnu/libc.so.6 (0x00000020232ca000)
```
3. check if the library libc.so offer the interface related with 'iconv'

```
root@zone:/opt/apps/zone/program# readelf -a /lib/aarch64-linux-gnu/libc.so.6 | grep iconv
  1636: 0000000000024a58    60 FUNC    GLOBAL DEFAULT   13 iconv_close@@GLIBC_2.17
  1833: 0000000000024830   548 FUNC    GLOBAL DEFAULT   13 iconv@@GLIBC_2.17
  1852: 0000000000024548   740 FUNC    GLOBAL DEFAULT   13 iconv_open@@GLIBC_2.17
```
ok,Bingo.
so the execute file docker use the iconv interface in libc.so,Not in libiconv.so;

```
root@zone:/opt/apps/zone/program# readelf -a ./libiconv.so | grep iconv
 0x000000000000000e (SONAME)             Library soname: [libiconv.so.2]
    26: 000000000002decc    92 FUNC    GLOBAL DEFAULT   11 libiconv_set_relocation_p
    27: 000000000002c3d0  1916 FUNC    GLOBAL DEFAULT   11 libiconv_open
    28: 000000000002da08   704 FUNC    GLOBAL DEFAULT   11 iconv_canonicalize
    29: 000000000002d378   684 FUNC    GLOBAL DEFAULT   11 libiconvctl
    30: 000000000002cb4c   196 FUNC    GLOBAL DEFAULT   11 libiconv
    31: 000000000002d7b8   592 FUNC    GLOBAL DEFAULT   11 libiconvlist
    32: 000000000002cc74  1796 FUNC    GLOBAL DEFAULT   11 libiconv_open_into
    33: 000000000010b0a8     4 OBJECT  GLOBAL DEFAULT   22 _libiconv_version
    34: 000000000002cc10   100 FUNC    GLOBAL DEFAULT   11 libiconv_close
    53: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS iconv.c
  1038: 000000000002df28   452 FUNC    LOCAL  DEFAULT   11 libiconv_relocate
  1043: 000000000002e0ec   136 FUNC    LOCAL  DEFAULT   11 libiconv_relocate2
  1054: 000000000002decc    92 FUNC    GLOBAL DEFAULT   11 libiconv_set_relocation_p
  1061: 000000000002cb4c   196 FUNC    GLOBAL DEFAULT   11 libiconv
  1066: 000000000002d7b8   592 FUNC    GLOBAL DEFAULT   11 libiconvlist
  1070: 000000000002c3d0  1916 FUNC    GLOBAL DEFAULT   11 libiconv_open
  1071: 000000000002da08   704 FUNC    GLOBAL DEFAULT   11 iconv_canonicalize
  1074: 000000000002cc10   100 FUNC    GLOBAL DEFAULT   11 libiconv_close
  1076: 000000000002cc74  1796 FUNC    GLOBAL DEFAULT   11 libiconv_open_into
  1077: 000000000002d378   684 FUNC    GLOBAL DEFAULT   11 libiconvctl
  1078: 000000000010b0a8     4 OBJECT  GLOBAL DEFAULT   22 _libiconv_version
```
4. Why?
in makefile ,we have add iconv include dir and lib dir ,why link the wrong target not the configured target;

ehhhhhh..

for -I$(ICONV_INC), the target dir is empty or removed,so cannot locate the header file 'icon.h' in folder 
$(ICONV_INC) ; 
the project compile ok ,because the makefile locate the iconv.h in systerm path '/user/include', the file in systerm path
is different from the configure path $(ICONV_INC);

5. here is the solution:
check the configure path $(ICONV_INC) in makefile,make the path valid;
the compile again,then check the depency with 'ldd' ,we get the following information:
```
root@zone:/opt/apps/zone# ldd ./program/dockertext
        linux-vdso.so.1 (0x0000007f8e5e5000)
        libiconv.so.2 => /opt/apps/zone/program/libiconv.so.2 (0x0000007f8e493000)
        libstdc++.so.6 => /lib/aarch64-linux-gnu/libstdc++.so.6 (0x0000007f8e28f000)
        libgcc_s.so.1 => /lib/aarch64-linux-gnu/libgcc_s.so.1 (0x0000007f8e26b000)
        libc.so.6 => /lib/aarch64-linux-gnu/libc.so.6 (0x0000007f8e0f8000)
        /lib/ld-linux-aarch64.so.1 (0x0000007f8e5b5000)
        libm.so.6 => /opt/apps/zone/program/libm.so.6 (0x0000007f8e04d000)

```

ok, we find the dependcy libiconv.so;next we can debug it well;


# 101 iconv library wrapper

```
std::string str = "中文";
IConvt convt("UTF-8", "GB18030");
convt.Convert(str, str);

IConvt cvt("GB18030", "UTF-8");
cvt.Convert(str, str);

//// iconv library wrapper

class IConvt
{
private:
	iconv_t m_handle;
	const size_t m_buffsize;
public:
	IConvt(const char* out_encode, const char* in_encode, size_t buf_size = 1024) :
		m_buffsize(buf_size)
	{
		m_handle = ::iconv_open(out_encode, in_encode);
	}

	~IConvt()
	{
		if (m_handle != (iconv_t)(-1))
			iconv_close(m_handle);
	}

	//解决特殊字符0x80（€）字符编码转换失败的问题。
	//参考Gdal代码的逻辑，实现字符编码的转换，当iconv的返回值不等于0时，表示源字符串转换到目的字符串的过程中，遇到
	//某一个字符转换失败（目前只知道0x80这个字符会转换失败），现在逻辑修改为跳过转换失败的字符，保证其余字符转换成功。

	//针对0x80（€）这个特殊的字符，是可以对应为相应的中文字符，当它为中文字符的时候，它由两个字节组成（0xA2E3），这是
	//它在GB18030下的编码，当它转换为Unicode编码时，它由三个字节组成，0xe282ac。所以这里特殊处理该字符。

	//0x80字符与（0xA2E3）表示的中文字符，本质上都表示欧元符号，但是显示并不是完全一样。在符号显示较小时，
	//0x80字符还是可以清晰地看到中间有两个横线，但是（0xA2E3）中间只有一根横线。当符号显示放大时，两个符号的
	//的显示又完全一样，中间都有两个横线。

	void Convert(const std::string& input, std::string& output) const
	{
		if (m_handle == (iconv_t)(-1))
			return;

		std::vector<char> in_buf(input.begin(), input.end());
		char* src_ptr = &in_buf[0];
		size_t src_size = input.size();

		std::vector<char> buf(m_buffsize);
		std::string dst;
		while (0 < src_size)
		{
			char* dst_ptr = &buf[0];
			size_t dst_size = buf.size();
			size_t res = ::iconv(m_handle, &src_ptr, &src_size, &dst_ptr, &dst_size);
			if (res == (size_t)-1)
			{
				if (errno == EILSEQ)
				{
					if (src_ptr[0] == '€') /// for case 0x80（€）
					{
						dst_ptr[0] = 0xe2;
						dst_ptr[1] = 0x82;
						dst_ptr[2] = 0xac;
						dst_ptr += 3;
						dst_size -= 3;
					}
					++src_ptr;
					--src_size;
				}
				else if (errno == E2BIG)
				{
					// 一批一批的转换
				/*	size_t nTmp = dst_size;
					dst_size *= 2;
					if (outbuf)
					{
						delete[] outbuf;
						outbuf = NULL;
					}
					outbuf = new char[dst_size + 1];
					memset(outbuf, 0, dst_size + 1);
					pszDstBuf = outbuf + nTmp - outlen;
					outlen += nTmp; */
				}
				else
				{
					break;
				}
			}
			dst.append(&buf[0], buf.size() - dst_size);
		}
		dst.swap(output);
	}
};

```

# 102 MultiByte2WideChar and WideChar2MultiByte

```
int  MultiByte2WideChar(const string &strSrc, wstring &strWdst)
{
	if (strSrc.empty())
	{
		strWdst = L"";
		return 1;
	}

	int inLen = strSrc.length();
	int outLen = sizeof(wchar_t) * (inLen + 1);
	wchar_t* pszTemp = new wchar_t[inLen + 1];
	memset(pszTemp, 0, outLen);
	std::string output;

#ifndef _WIN32
	IConvt convt("UTF-32LE", "GB18030");
#else
	IConvt convt("UTF-16LE", "GB18030");
#endif
	convt.Convert(strSrc, output);
	memcpy(pszTemp,output.c_str(),output.size());
	strWdst = pszTemp;
	if (pszTemp)
	{
		delete[] pszTemp;
		pszTemp = NULL;
	}
	return 1;
}

//==========================WideChar2MultiByte()==========================
// 宽字节转多字节
//
// @param [in] strWsrc 传入的宽字节
// @param [out] strDst 返出的多字节
//
// @return 成功返回1，失败返回0
//
// @remark 为避免不同操作系统下可能无法找到正确的字符集的问题，现在统一使用以下接口进行多字节转宽字节
// setlocale()耗时较多，依照GBKToUTF8()使用iconv重新实现此函数，
//			   可减少大约75%耗时。
//================================================================================
int  WideChar2MultiByte(const wstring &strWsrc, string &strDst) 
{
	if (strWsrc.empty())
	{
		strDst = "";
		return 1;
	}
	int inLen = strWsrc.length();
	int outLen = (inLen + 1) * 4 ;
	inLen = inLen * sizeof(wchar_t);

 #ifndef _WIN32
	IConvt convt("GB18030"，"UTF-32LE");
 #else
	IConvt convt("GB18030"，"UTF-16LE");
#endif
	std::string output;
	convt.Convert(strSrc, strDst);
	return 1;
}

```


```
//=================================code_convert()=================================
// 封装iconv第三方库实现的用于字符编码转换的接口
//
// @param [in] from_charset 源字符串编码方式，可传入GBK，UTF-8
// @param [in] to_charset   目的字符串编码方式，可传入GBK，UTF-8
// @param [in] inbuf		源字符指针
// @param [in] inlen		源字符串长度，最多读取源字符串的inbuf个字符进行转换
// @param [out] outbuf		目的字符串指针
// @param [in] outlen		目的字符串长度，最多转换outlen个字符到目的字符串
//
// @return 成功返回1，失败返回0
//================================================================================
int code_convert(char *from_charset, char *to_charset, char *inbuf, size_t inlen, char *outbuf, size_t outlen)
{

}
int  MultiByte2WideChar(const string &strSrc, wstring &strWdst)
{
	if (strSrc.empty())
	{
		strWdst = L"";
		return 1;
	}

	int inLen = strSrc.length();
	int outLen = sizeof(wchar_t) * (inLen + 1);
	wchar_t* pszTemp = new wchar_t[inLen + 1];
	memset(pszTemp, 0, outLen);
#ifndef _WIN32

	if (1 != code_convert("GB18030", "UTF-32LE", (char *)strSrc.c_str(), inLen, (char *)pszTemp, outLen))
#else
	if (1 != code_convert("GB18030", "UTF-16LE", (char *)strSrc.c_str(), inLen, (char *)pszTemp, outLen))
#endif
	{
		strWdst = L"";
		if (pszTemp)
		{
			delete[] pszTemp;
			pszTemp = NULL;
		}
		return 0;
	}
	strWdst = pszTemp;
	if (pszTemp)
	{
		delete[] pszTemp;
		pszTemp = NULL;
	}
	return 1;
}

//==========================WideChar2MultiByte()==========================
// 宽字节转多字节
//
// @param [in] strWsrc 传入的宽字节
// @param [out] strDst 返出的多字节
//
// @return 成功返回1，失败返回0
//
// @remark 为避免不同操作系统下可能无法找到正确的字符集的问题，现在统一使用以下接口进行多字节转宽字节
// setlocale()耗时较多，依照GBKToUTF8()使用iconv重新实现此函数，
//			   可减少大约75%耗时。
//================================================================================
int  WideChar2MultiByte(const wstring &strWsrc, string &strDst) 
{
	if (strWsrc.empty())
	{
		strDst = "";
		return 1;
	}
	int inLen = strWsrc.length();
	int outLen = (inLen + 1) * 4 ;
	inLen = inLen * sizeof(wchar_t);

	char* pszTemp = new char[outLen];
	memset(pszTemp, 0, outLen);
 #ifndef _WIN32
	if (1 != code_convert("UTF-32LE", "GB18030", (char *)strWsrc.c_str(), inLen, pszTemp, outLen))
 #else
	if (1 != code_convert("UTF-16LE", "GB18030", (char *)strWsrc.c_str(), inLen, pszTemp, outLen))
#endif
	{
		strDst = "";
		if (pszTemp)
		{
			delete[] pszTemp;
			pszTemp = NULL;
		}
		return 0;
	}
	strDst = pszTemp;
	if (pszTemp)
	{
		delete[] pszTemp;
		pszTemp = NULL;
	}
	return 1;
}
```

# 103 fonts.conf

Fonfconfig的配置文件采用了模块化的结构。配置文件由以下文件组成
```
/usr/local/etc/fonts/fonts.conf
/usr/local/etc/fonts/conf.avail/*.conf
/usr/local/etc/fonts/conf.d/*.conf
```
/usr/local/etc/fonts/conf.d/ 目录下的文件大多数是 conf.avail/ 目录下的连接，大致是如下这些：
```
20-fix-globaladvance.conf
20-lohit-gujarati.conf
20-unhint-small-vera.conf
30-amt-aliases.conf
30-cjk-aliases.conf
40-generic.conf
49-sansserif.conf
50-user.conf
51-local.conf
60-latin.conf
65-fonts-persian.conf
65-nonlatin.conf
69-language-selector-zh-cn.conf
69-unifont.conf
80-delicious.conf
90-synthetic.conf
```
前面的数字用来控制执行的先后顺序，从名称上就可以看出，每个.conf文件都有针对性的字体特性进行处理。

而实现这种模块化，所借助的就是 /usr/local/etc/fonts/fonts.conf 文件。


## /etc/fonts/fonts.conf
```
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<!-- /etc/fonts/fonts.conf file to configure system font access -->
<fontconfig>
        <its:rules xmlns:its="http://www.w3.org/2005/11/its" version="1.0">
                <its:translateRule translate="no" selector="/fontconfig/*[not(self::description)]"/>
        </its:rules>

        <description>Default configuration file</description>

<!--
        DO NOT EDIT THIS FILE.
        IT WILL BE REPLACED WHEN FONTCONFIG IS UPDATED.
        LOCAL CHANGES BELONG IN 'local.conf'.

        The intent of this standard configuration file is to be adequate for
        most environments.  If you have a reasonably normal environment and
        have found problems with this configuration, they are probably
        things that others will also want fixed.  Please submit any
        problems to the fontconfig bugzilla system located at fontconfig.org

        Note that the normal 'make install' procedure for fontconfig is to
        replace any existing fonts.conf file with the new version.  Place
        any local customizations in local.conf which this file references.

        Keith Packard
-->

<!-- Font directory list -->

        <dir>/usr/share/fonts</dir>
        <dir>/usr/local/share/fonts</dir>
        <dir prefix="xdg">fonts</dir>
        <!-- the following element will be removed in the future -->
        <dir>~/.fonts</dir>

<!--
  Accept deprecated 'mono' alias, replacing it with 'monospace'
-->
        <match target="pattern">
                <test qual="any" name="family">
                        <string>mono</string>
                </test>
                <edit name="family" mode="assign" binding="same">
                        <string>monospace</string>
                </edit>
        </match>

<!--
  Accept alternate 'sans serif' spelling, replacing it with 'sans-serif'
-->
        <match target="pattern">
                <test qual="any" name="family">
                        <string>sans serif</string>
                </test>
                <edit name="family" mode="assign" binding="same">
                        <string>sans-serif</string>
                </edit>
        </match>

<!--
  Accept deprecated 'sans' alias, replacing it with 'sans-serif'
-->
        <match target="pattern">
                <test qual="any" name="family">
                        <string>sans</string>
                </test>
                <edit name="family" mode="assign" binding="same">
                        <string>sans-serif</string>
                </edit>
        </match>

<!--
  Ignore dpkg temporary files created in fonts directories
-->
        <selectfont>
                <rejectfont>
                        <glob>*.dpkg-tmp</glob>
                </rejectfont>
        </selectfont>
        <selectfont>
                <rejectfont>
                        <glob>*.dpkg-new</glob>
                </rejectfont>
        </selectfont>

<!--
  Load local system customization file
-->
        <include ignore_missing="yes">conf.d</include>

<!-- Font cache directory list -->

        <cachedir>/var/cache/fontconfig</cachedir>
        <cachedir prefix="xdg">fontconfig</cachedir>
        <!-- the following element will be removed in the future -->
        <cachedir>~/.fontconfig</cachedir>

        <config>
<!--
  Rescan configuration every 30 seconds when FcFontSetList is called
 -->
                <rescan>
                        <int>30</int>
                </rescan>
        </config>

</fontconfig>

```

## /etc/fonts/local.conf
/// another example

```
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <its:rules xmlns:its="http://www.w3.org/2005/11/its" version="1.0">
    <its:translateRule
      translate="no"
      selector="/fontconfig/*[not(self::description)]"
    />
  </its:rules>

  <description>Android Font Config</description>

  <!-- Font directory list -->

  <dir>/usr/share/fonts</dir>
  <dir>/usr/local/share/fonts</dir>
  <dir prefix="xdg">fonts</dir>
  <!-- the following element will be removed in the future -->
  <dir>~/.fonts</dir>

  <!-- 关闭内嵌点阵字体 -->
  <match target="font">
    <edit name="embeddedbitmap" mode="assign">
      <bool>false</bool>
    </edit>
  </match>

  <!-- 英文默认字体使用 Roboto 和 Noto Serif ,终端使用 DejaVu Sans Mono. -->
  <match>
    <test qual="any" name="family">
      <string>serif</string>
    </test>
    <edit name="family" mode="prepend" binding="strong">
      <string>Noto Serif</string>
    </edit>
  </match>
  <match target="pattern">
    <test qual="any" name="family">
      <string>sans-serif</string>
    </test>
    <edit name="family" mode="prepend" binding="strong">
      <string>Roboto</string>
    </edit>
  </match>
  <match target="pattern">
    <test qual="any" name="family">
      <string>monospace</string>
    </test>
    <edit name="family" mode="prepend" binding="strong">
      <string>DejaVu Sans Mono</string>
    </edit>
  </match>

  <!-- 中文默认字体使用思源黑体和思源宋体,不使用　Noto Sans CJK SC 是因为这个字体会在特定情况下显示片假字. -->
  <match>
    <test name="lang" compare="contains">
      <string>zh</string>
    </test>
    <test name="family">
      <string>serif</string>
    </test>
    <edit name="family" mode="prepend">
      <string>Source Han Serif CN</string>
    </edit>
  </match>
  <match>
    <test name="lang" compare="contains">
      <string>zh</string>
    </test>
    <test name="family">
      <string>sans-serif</string>
    </test>
    <edit name="family" mode="prepend">
      <string>Source Han Sans CN</string>
    </edit>
  </match>
  <match>
    <test name="lang" compare="contains">
      <string>zh</string>
    </test>
    <test name="family">
      <string>monospace</string>
    </test>
    <edit name="family" mode="prepend">
      <string>Noto Sans Mono CJK SC</string>
    </edit>
  </match>

  <!-- Windows & Linux Chinese fonts. -->
  <!-- 把所有常见的中文字体映射到思源黑体和思源宋体，这样当这些字体未安装时会使用思源黑体和思源宋体.
解决特定程序指定使用某字体，并且在字体不存在情况下不会使用fallback字体导致中文显示不正常的情况. -->
  <match target="pattern">
    <test qual="any" name="family">
      <string>WenQuanYi Zen Hei</string>
    </test>
    <edit name="family" mode="assign" binding="same">
      <string>Source Han Sans CN</string>
    </edit>
  </match>
  <match target="pattern">
    <test qual="any" name="family">
      <string>WenQuanYi Micro Hei</string>
    </test>
    <edit name="family" mode="assign" binding="same">
      <string>Source Han Sans CN</string>
    </edit>
  </match>
  <match target="pattern">
    <test qual="any" name="family">
      <string>WenQuanYi Micro Hei Light</string>
    </test>
    <edit name="family" mode="assign" binding="same">
      <string>Source Han Sans CN</string>
    </edit>
  </match>
  <match target="pattern">
    <test qual="any" name="family">
      <string>Microsoft YaHei</string>
    </test>
    <edit name="family" mode="assign" binding="same">
      <string>Source Han Sans CN</string>
    </edit>
  </match>
  <match target="pattern">
    <test qual="any" name="family">
      <string>SimHei</string>
    </test>
    <edit name="family" mode="assign" binding="same">
      <string>Source Han Sans CN</string>
    </edit>
  </match>
  <match target="pattern">
    <test qual="any" name="family">
      <string>SimSun</string>
    </test>
    <edit name="family" mode="assign" binding="same">
      <string>Source Han Serif CN</string>
    </edit>
  </match>
  <match target="pattern">
    <test qual="any" name="family">
      <string>SimSun-18030</string>
    </test>
    <edit name="family" mode="assign" binding="same">
      <string>Source Han Serif CN</string>
    </edit>
  </match>

  <!-- Load local system customization file -->
  <include ignore_missing="yes">conf.d</include>

  <!-- Font cache directory list -->

  <cachedir>/var/cache/fontconfig</cachedir>
  <cachedir prefix="xdg">fontconfig</cachedir>
  <!-- the following element will be removed in the future -->
  <cachedir>~/.fontconfig</cachedir>

  <config>
    <!-- Rescan configuration every 30 seconds when FcFontSetList is called -->
    <rescan>
      <int>30</int>
    </rescan>
  </config>
</fontconfig>
```
## /etc/fonts/conf.d/30-cjk-aliases.conf

```
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
<!-- Aliases for Korean fonts -->
    <alias>
        <family>Batang</family>
        <accept>
	    <family>Noto Serif CJK KR</family>
	    <family>NanumMyeongjo</family>
            <family>UnBatang</family>
        </accept>
    </alias>
    <alias>
        <family>바탕</family>
        <accept>
	    <family>Noto Serif CJK KR</family>
	    <family>NanumMyeongjo</family>
            <family>UnBatang</family>
        </accept>
    </alias>
    <alias>
        <family>궁서</family>
        <accept>
	    <family>Noto Serif CJK KR</family>
            <family>UnGungseo</family> 
	    <family>NanumMyeongjo</family>
        </accept>
    </alias>
    <alias>
        <family>GungsuhChe</family>
        <accept>
	    <family>Noto Serif CJK KR</family>
            <family>UnGungseo</family> 
	    <family>NanumMyeongjo</family>
        </accept>
    </alias>
    <alias>
        <family>궁서체</family>
        <accept>
	    <family>Noto Serif CJK KR</family>
            <family>UnGungseo</family> 
	    <family>NanumMyeongjo</family>
        </accept>
    </alias>
<!-- Aliases for Japanese Windows fonts -->
    <alias>
        <family>MS Gothic</family>
        <accept>
            <family>Noto Sans Mono CJK JP</family>
            <family>TakaoGothic</family>
            <family>IPAGothic</family>
            <family>IPAMonaGothic</family>
            <family>VL Gothic</family>
            <family>Sazanami Gothic</family>
            <family>Kochi Gothic</family>
        </accept>
    </alias>
    <alias>
        <family>ＭＳ ゴシック</family>
        <accept>
            <family>Noto Sans Mono CJK JP</family>
            <family>TakaoGothic</family>
            <family>IPAGothic</family>
            <family>IPAMonaGothic</family>
            <family>VL Gothic</family>
            <family>Sazanami Gothic</family>
            <family>Kochi Gothic</family>
        </accept>
    </alias>
    
    <alias>
        <family>メイリオ</family>
        <accept>
            <family>IPAexGothic</family>
        </accept>
    </alias>
<!-- Aliases for Simplified Chinese Windows fonts -->
    <alias>
        <family>SimSun</family>
        <accept>
            <family>Noto Serif CJK SC</family>
            <family>HYSong</family>
            <family>AR PL UMing CN</family>
        </accept>
    </alias>
    <alias>
        <family>NSimSun</family>
        <accept>
            <family>Noto Serif CJK SC</family>
            <family>HYSong</family>
            <family>AR PL UMing CN</family>
        </accept>
    </alias>
    <alias>
        <family>宋体</family>
        <accept>
            <family>Noto Serif CJK SC</family>
            <family>HYSong</family>
            <family>AR PL UMing CN</family>
        </accept>
    </alias>
    <alias>
        <family>新宋体</family>
        <accept>
            <family>Noto Serif CJK SC</family>
            <family>HYSong</family>
            <family>AR PL UMing CN</family>
        </accept>
    </alias>
    <alias>
        <family>KaiTi</family>
        <accept>
            <family>Noto Serif CJK SC</family>
            <family>AR PL UKai CN</family>
            <family>AR PL ZenKai Uni</family>
        </accept>
    </alias>
    <alias>
        <family>楷体</family>
        <accept>
            <family>Noto Serif CJK SC</family>
            <family>AR PL UKai CN</family>
            <family>AR PL ZenKai Uni</family>
        </accept>
    </alias>
    <alias>
        <family>Microsoft YaHei</family>
        <accept>
            <family>Noto Sans CJK SC</family>
            <family>WenQuanYi Micro Hei</family>
            <family>WenQuanYi Zen Hei</family>
        </accept>
    </alias>
    <alias>
        <family>微软雅黑</family>
        <accept>
            <family>Noto Sans CJK SC</family>
            <family>WenQuanYi Micro Hei</family>
            <family>WenQuanYi Zen Hei</family>
        </accept>
    </alias>
<!-- Aliases for Traditional Chinese Windows fonts -->
    <alias>
        <family>MingLiU</family>
        <accept>
            <family>Noto Serif CJK TC</family>
            <family>AR PL UMing TW</family>
        </accept>
    </alias>
    <alias>
        <family>細明體</family>
        <accept>
            <family>Noto Serif CJK TC</family>
            <family>AR PL UMing TW</family>
        </accept>
    </alias>
    <alias>
        <family>PMingLiU</family>
        <accept>
            <family>Noto Serif CJK TC</family>
            <family>AR PL UMing TW</family>
        </accept>
    </alias>
    <alias>
        <family>新細明體</family>
        <accept>
            <family>Noto Serif CJK TC</family>
            <family>AR PL UMing TW</family>
        </accept>
    </alias>
    <alias>
        <family>微軟正黑體</family>
        <accept>
            <family>Noto Sans CJK TC</family>
            <family>WenQuanYi Micro Hei</family>
            <family>WenQuanYi Zen Hei</family>
        </accept>
    </alias>
<!-- Alias for HKSCS -->
    <alias>
        <family>Ming (for ISO10646)</family>
        <accept>
            <family>AR PL UMing HK</family>
        </accept>
    </alias>
    <alias>
        <family>MingLiU_HKSCS</family>
        <accept>
            <family>AR PL UMing HK</family>
        </accept>
    </alias>
    <alias>
        <family>細明體_HKSCS</family>
        <accept>
            <family>AR PL UMing HK</family>
        </accept>
    </alias>
</fontconfig>
```


# 104 font type serif sans-serif monospace


[Linux桌面系统字体配置详解](http://www.gimoo.net/t/1504/55322b241ea93.html)

```
　　1、英文字体分为三类，分别是有衬线字体（serif）、无衬线字体（sans-serif）和等宽字体（monospace）。Serif是有衬线字体，意思是在字的笔画开始、结束的地方有额外的装饰，而且笔画的粗细会有所不同。Sans-serif就没有这些额外的装饰，而且笔画的粗细差不多。在传统的正文印刷中，普遍认为衬线体能带来更佳的可读性（相比无衬线体），尤其是在大段落的文章中，衬线增加了阅读时对字母的视觉参照。而无衬线体往往被用在标题、较短的文字段落或者一些通俗读物中。相比严肃正经的衬线体，无衬线体给人一种休闲轻松的感觉。同时，由于无衬线字体笔画比较饱满，所以比较适合电脑屏幕显示，在印刷和打印中，可以用无衬线字体做标题、加粗字体等表示强调。等宽字体就不用多说啦，主要用于终端字体或编程。

　　2、中文字体可以参照英文字体进行分类，由于中文都是等宽的，所以就只需要区分有衬线（serif）和无衬线（sans-serif）。中文的宋体、仿宋就相当于英文的serif，所以用于传统印刷和打印效果比较好。而中文的黑体、楷体、圆体等字体相当于英文的sans-serif，用于电脑屏幕的显示效果比较好，也可以用在印刷和打印中做标题和粗体字。

　　3、Serif字体的经典代表有Georgia和Times New Roman，sans-serif字体的经典代表有Arial和Verdana，monospace字体的经典代表有Courier New和DejaVu Sans Mono。
```

## /etc/fonts/conf.d/69-language-selector-zh-cn.conf
```
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>

	<match target="pattern">
        <test name="lang">
            <string>zh-cn</string>
        </test>
		<test qual="any" name="family">
			<string>serif</string>
		</test>
		<edit name="family" mode="prepend" binding="strong">
			<string>Noto Serif CJK SC</string>
			<string>HYSong</string>
			<string>AR PL UMing CN</string>
			<string>AR PL UMing HK</string>
			<string>AR PL New Sung</string>
			<string>WenQuanYi Bitmap Song</string>
			<string>AR PL UKai CN</string>
			<string>AR PL ZenKai Uni</string>
		</edit>
	</match> 
	<match target="pattern">
		<test qual="any" name="family">
			<string>sans-serif</string>
		</test>
        <test name="lang">
            <string>zh-cn</string>
        </test>
		<edit name="family" mode="prepend" binding="strong">
			<string>Noto Sans CJK SC</string>
			<string>WenQuanYi Zen Hei</string>
			<string>HYSong</string>
			<string>AR PL UMing CN</string>
			<string>AR PL UMing HK</string>
			<string>AR PL New Sung</string>
			<string>AR PL UKai CN</string>
			<string>AR PL ZenKai Uni</string>
		</edit>
	</match> 
	<match target="pattern">
		<test qual="any" name="family">
			<string>monospace</string>
		</test>
        <test name="lang">
            <string>zh-cn</string>
        </test>
		<edit name="family" mode="prepend" binding="strong">
			<string>DejaVu Sans Mono</string>
			<string>Noto Sans Mono CJK SC</string>
			<string>WenQuanYi Zen Hei Mono</string>
			<string>HYSong</string>
			<string>AR PL UMing CN</string>
			<string>AR PL UMing HK</string>
			<string>AR PL New Sung</string>
			<string>AR PL UKai CN</string>
			<string>AR PL ZenKai Uni</string>
		</edit>
	</match> 

</fontconfig>
```

# 105 compile Qt which support ttf

1.编译QT库时需要支持TTF字体
```
./configure -qt-freetype -fontconfig ...

./configure -static -release -prefix ~/qt-static/install -opensource -confirm-license -no-compile-examples -nomake examples -no-openssl -no-libpng -qt-freetype  -fontconfig

```
```
Qt源码中包含了一些第三方库，如果想使用Qt自带的第三方库，可用通过-qt配置；
如果想使用系统中的第三方库，可用通过-system配置。下表中列出一些第三方库及其配置选项。

Library Name             Bundled in Qt                         Installed in System
zlib                    -qt-zlib​​                               -system-zlib​​
libjpeg                 -qt-libjpeg​​                            -system-libjpeg​​
libpng                  ​-qt-libpng​​                   ​​          -system-libpng​​
xcb                  ​​   -qt-xcb​​                                ​​-system-xcb​​
xkbcommon               ​​-qt-xkbcommon​​                          ​​-system-xkbcommon​​
freetype                ​​-qt-freetype​​                           ​​-system-freetype​​
PCRE                    ​​-qt-pcre​​                      ​​         -system-pcre​​
HarfBuzz-NG             ​​-qt-harfbuzz​​                    ​​       -system-harfbuzz​​

当然，也可以禁用这些第三方库，用-no替换-qt就行，如下所示。
```

# 106 copyright

```
Copyright © 2000,2001,2002,2003,2004,2006,2007 Keith Packard
Copyright © 2005 Patrick Lam
Copyright © 2009 Roozbeh Pournader
Copyright © 2008,2009 Red Hat, Inc.
Copyright © 2008 Danilo Šegan
Copyright © 2012 Google, Inc.


Permission to use, copy, modify, distribute, and sell this software and its
documentation for any purpose is hereby granted without fee, provided that
the above copyright notice appear in all copies and that both that
copyright notice and this permission notice appear in supporting
documentation, and that the name of the author(s) not be used in
advertising or publicity pertaining to distribution of the software without
specific, written prior permission.  The authors make no
representations about the suitability of this software for any purpose.  It
is provided "as is" without express or implied warranty.

THE AUTHOR(S) DISCLAIMS ALL WARRANTIES WITH REGARD TO THIS SOFTWARE,
INCLUDING ALL IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS, IN NO
EVENT SHALL THE AUTHOR(S) BE LIABLE FOR ANY SPECIAL, INDIRECT OR
CONSEQUENTIAL DAMAGES OR ANY DAMAGES WHATSOEVER RESULTING FROM LOSS OF USE,
DATA OR PROFITS, WHETHER IN AN ACTION OF CONTRACT, NEGLIGENCE OR OTHER
TORTIOUS ACTION, ARISING OUT OF OR IN CONNECTION WITH THE USE OR
PERFORMANCE OF THIS SOFTWARE.

```
# 107 SceneBuilder auto generate Controller Skeleton

1. Open SceneBuilder, create the fxml file;
2. name the actions and the control items;
3. link the fxml file with controller file;such as
   fx:controller="com.javafx.fxml.LoginController"
4. In SceneBuilder, in view menu, click 'view–>Show Sample Controller Skeleton';
   we will get the Controller Skeleton,such as

  
``` 
import javafx.fxml.FXML;
import javafx.scene.control.Hyperlink;
import javafx.scene.control.Label;
import javafx.scene.layout.HBox;

public class AboutController {

    @FXML
    private Label version;

    @FXML
    private HBox newVersionBox;

    @FXML
    private Hyperlink archivesLink;

    @FXML
    private Hyperlink sourceCodeLink;

    @FXML
    private Hyperlink licenseLink;


    @FXML
    void browseArchives(ActionEvent event) {

    }

    @FXML
    void browseChangeLog(ActionEvent event) {

    }

}

```

# 108 configure user in linux

```
1. 从系统中添加组
 groupadd group2
2.  从系统中删除组
 groupdel group2  
 
3.# groupmod -g 102 group2
此命令将组group2的组标识号改动为102。
# groupmod –g 10000 -n group3 group2
此命令将组group2的标识号改为10000，组名改动为group3。

4.useradd 选项 username
当中各选项含义例如以下：

代码:
-c comment 指定一段凝视性描写叙述。
-d 文件夹 指定用户主文件夹，假设此文件夹不存在，则同一时候使用-m选项，能够创建主文件夹。
-g 用户组 指定用户所属的用户组。
-G 用户组，用户组 指定用户所属的附加组。
-s Shell文件 指定用户的登录Shell。
-u 用户号 指定用户的用户号，假设同一时候有-o选项，则能够反复使用其它用户的标识号。

username 指定新账号的登录名

useradd  -d /home/data/ -m data -g group2

# useradd –d /usr/sam -m sam
 
此命令创建了一个用户sam，
当中-d和-m选项用来为登录名sam产生一个主文件夹/usr/sam（/usr为默认的用户主文件夹所在的父文件夹）

useradd -s /bin/sh -g group –G adm,root gem
此命令新建了一个用户gem，该用户的登录Shell是/bin/sh，它属于group用户组，同一时候又属于adm和root用户组，当中group用户组是其主组。
这里可能新建组：#groupadd group及groupadd adm　
增加用户账号就是在/etc/passwd文件里为新用户增加一条记录，同一时候更新其它系统文件如/etc/shadow, /etc/group等

5. # userdel sam
此命令删除用户sam在系统文件里（主要是/etc/passwd, /etc/shadow, /etc/group等）的记录，同一时候删除用户的主文件夹。
# usermod -s /bin/ksh -d /home/z –g developer sam
此命令将用户sam的登录Shell改动为ksh，主文件夹改为/home/z，用户组改为developer。

6. 比如，假设当前用户是sam，则以下的命令改动该用户自己的口令：
$ passwd
Old password:******
New password:*******
Re-enter new password:*******

假设是超级用户，能够用下列形式指定不论什么用户的口令：
# passwd sam
New password:*******
Re-enter new password:*******


```

# 109 inflate buffer

```

#include "zlib.h" 
#define INTERNAL_BUFFER_SIZE 4096
class flate
{
public:
	flate();
	~flate();
protected:
	z_stream m_stream;
	unsigned char m_buffer[INTERNAL_BUFFER_SIZE];

public:
	/// decode
	void    BeginDecode();
	__int64 DecodeBlock(const char* pInBuffer, __int64 lInLen);
	void    EndDecode();

	/// encode
	void    BeginEncode();
	void    EncodeBlock(const char* pBuffer, __int64 lLen);
	void    EndEncode();

private:
	void    EncodeBlockInternal(const char* pBuffer, __int64 lLen, int nMode);
	void    FailEncodeDecode();

};

////////////////////////////////

flate::flate()
{

}

flate::~flate()
{

}

void flate::BeginDecode()
{
	m_stream.zalloc = Z_NULL;
	m_stream.zfree = Z_NULL;
	m_stream.opaque = Z_NULL;

	if (inflateInit(&m_stream) != Z_OK)
	{
		//RAISE_ERROR(eError_Flate);
	}
}

__int64 flate::DecodeBlock(const char* pInBuffer, __int64 lInLen)
{
	int flateErr;
	int nWrittenData;

	m_stream.avail_in = static_cast<long>(lInLen);
	m_stream.next_in = reinterpret_cast<Bytef*>(const_cast<char*>(pInBuffer));

	m_stream.avail_out = INTERNAL_BUFFER_SIZE;
	m_stream.next_out = m_buffer;

	switch ((flateErr = inflate(&m_stream, Z_NO_FLUSH))) {
	case Z_NEED_DICT:
	case Z_DATA_ERROR:
	case Z_MEM_ERROR:
	{
		//PdfError::LogMessage(eLogSeverity_Error, "Flate Decoding Error from ZLib: %i\n", flateErr);
		(void)inflateEnd(&m_stream);

		FailEncodeDecode();
		//RAISE_ERROR(ePdfError_Flate);
	}
	default:
		break;
	}
	nWrittenData = INTERNAL_BUFFER_SIZE - m_stream.avail_out;

	return nWrittenData;
}

void flate::EndDecode()
{
	(void)inflateEnd(&m_stream);
}

void flate::BeginEncode()
{
	m_stream.zalloc = Z_NULL;
	m_stream.zfree = Z_NULL;
	m_stream.opaque = Z_NULL;

	if (deflateInit(&m_stream, Z_DEFAULT_COMPRESSION))
	{
		//PODOFO_RAISE_ERROR(ePdfError_Flate);
	}
}

void flate::EncodeBlock(const char* pBuffer, __int64 lLen)
{
	this->EncodeBlockInternal(pBuffer, lLen, Z_NO_FLUSH);
}

void flate::EndEncode()
{
	this->EncodeBlockInternal(NULL, 0, Z_FINISH);
	deflateEnd(&m_stream);
}

void flate::EncodeBlockInternal(const char* pBuffer, __int64 lLen, int nMode)
{
	int nWrittenData = 0;

	m_stream.avail_in = static_cast<long>(lLen);
	m_stream.next_in = reinterpret_cast<Bytef*>(const_cast<char*>(pBuffer));

	do {
		m_stream.avail_out = INTERNAL_BUFFER_SIZE;
		m_stream.next_out = m_buffer;

		if (deflate(&m_stream, nMode) == Z_STREAM_ERROR)
		{
			FailEncodeDecode();
			///PODOFO_RAISE_ERROR(ePdfError_Flate);
		}


		nWrittenData = INTERNAL_BUFFER_SIZE - m_stream.avail_out;

	} while (m_stream.avail_out == 0);
}

void flate::FailEncodeDecode()
{

}


```


# 110 draw pdf With TilingPattern
	
```
void drawWithTilingPattern(PdfDocument* pParent, PdfXObject* pXObj, PdfPainter* pPainter)
{
	///pPainter->Save();
	double strokeR = 0.1, strokeG = 0.0, strokeB = 1.0;
	bool doFill = false; double fillR = 0.7, fillG = 0.7, fillB = 0.7;
	double offsetX = NUMPAT, offsetY = NUMPAT;
	if (0)
	{
		if (!pParent)  return;
		PdfImage pdfImage(pParent);
		EPdfTilingPatternType eTilingType = ePdfTilingPatternType_Image;
		pdfImage.LoadFromFile("F:/User/Pictures/icon-29-2x.png");
		double h = pdfImage.GetHeight();
		double w = pdfImage.GetWidth();
		offsetX = w*3, offsetY = h*3;
		PdfTilingPattern rPattern(eTilingType, strokeR, strokeG, strokeB, doFill, fillR, fillG, fillB,
			offsetX, offsetY, &pdfImage, pParent);
		pPainter->SetTilingPattern(rPattern);
	}
	else
	{
		
		PdfObject* pContent2 = pXObj->GetContents();
		PdfObject* pContent = pXObj->GetObject();
		PdfVecObjects* objs = pContent->GetOwner();

		PdfStream* pstream = pContent->GetStream();
		char* pBuffer = nullptr; pdf_long  lLen = 0;
		pstream->GetCopy(&pBuffer, &lLen);
		podofo_free(pBuffer);
		pstream->GetFilteredCopy(&pBuffer, &lLen);

		
		//parsePDFObject(pContent);
		PdfRect rct = pXObj->GetPageSize();
		EPdfTilingPatternType eTilingType = ePdfTilingPatternType_Cross;
		offsetX = 0; 
		offsetY = 0; 
		double dmi = 72 / 25.4;
		double xstep = rct.GetWidth()/ dmi;
		double ystep = rct.GetHeight()/ dmi;
		PdfTilingPattern rPattern(eTilingType, strokeR, strokeG, strokeB, doFill, fillR, fillG, fillB,
			offsetX, offsetY, nullptr, pContent->GetOwner());

		double scale = (xstep / 8)/(dmi);
		PdfArray array;
		array.push_back(static_cast<pdf_int64>(3));
		array.push_back(static_cast<pdf_int64>(0));
		array.push_back(static_cast<pdf_int64>(0));
		array.push_back(static_cast<pdf_int64>(3));
		array.push_back(offsetX);
		array.push_back(offsetY);
		rPattern.GetObject()->GetDictionary().AddKey(PdfName("Matrix"), array);
		//rPattern.GetObject()->GetDictionary().AddKey(PdfName("XStep"), static_cast<int64_t>(10));
		//rPattern.GetObject()->GetDictionary().AddKey(PdfName("YStep"), static_cast<int64_t>(10));


		TVecFilters vecFlate;

		if (0) {
			vecFlate.push_back(ePdfFilter_FlateDecode);
			std::ostringstream out; fillB = 1.0;
			out << fillR << " " << fillG << " " << fillB << " rg" << " ";
			out << 0 << " " << 0 << " " << 30 << " " << 20 << " re" << " ";
			out << "f" << " ";

			std::string str = out.str();
			PdfMemoryInputStream stream(str.c_str(), str.length());
			rPattern.GetObject()->GetStream()->Set(&stream, vecFlate);
		}
		else {
			vecFlate.push_back(ePdfFilter_FlateDecode);
			PdfMemoryInputStream stream(pBuffer, lLen);
			rPattern.GetObject()->GetStream()->Set(&stream, vecFlate);
		    podofo_free(pBuffer);
		}

		pPainter->SetTilingPattern(rPattern);
	//	initPattern(rct.GetWidth(), rct.GetHeight(), pXObj);


	}

	/// fill the tiling pattern
	{
		const double     x = 10000 * CONVERSION_CONSTANT;
		const double     y = 260000 * CONVERSION_CONSTANT; // 27cm
		const double   dWidth = 180000 * CONVERSION_CONSTANT; // 18cm
		size_t size = floor(dWidth / NUMPAT);
		double ybase = y - NUMPAT*(size - 2) / 2;
		double xmid = (x + NUMPAT*(size - 2) / 2 + x)*0.5;
		int ncount = 4;
		POINT dots[4] = { x,ybase,NUMPAT*(size - 1) + x,ybase,xmid,y - dWidth*0.8,x + (x + xmid)*0.1,(ybase + y - dWidth)*0.5 };
		i_AddPath(pPainter, dots, ncount);

		pPainter->ClosePath();
		pPainter->Fill();

	//	pPainter->Clip(true);
	}
	//pPainter->Restore();
}
	
```

# 111 draw pdf With TilingPattern


```
#include <cstdlib>
#include <cstdio>
#include <iostream>
#include <podofo.h>

typedef struct tagD_Point
{
	double x;
	double y;
} 	D_Point;

class TilePattern
{
public:
	TilePattern();
	~TilePattern();

public:
	void TestTilePattern();

	void drawWithTilingPattern(PdfDocument* pParent, PdfXObject* pXObj, PdfPainter* pPainter);

	void Save(const char* pszFile);
protected:

	void initPattern(int patWid, int patHei, PdfXObject* pXObj, PdfDocument* pParent);

	PdfXObject* createPattern(int patWid, int patHei, PdfDocument* pParent, PdfPainter* pPainter);
	
	void SetMatrix(PdfTilingPattern& rPattern, double xscale, double yscale, double xoffset, double yoffset);

	int i_AddPath(PdfPainter* painter, D_Point* dots, int count);

	void drawWithXobject(PdfXObject* pXObj, PdfPainter* pPainter);
	/// draw polyline with different style 
	void drawPolyline();
	/// fill back color
	void FillBkGround();
private:
	PdfPainter* m_pPainter;
	PdfMemDocument doc;
	PdfPage*pPage;
	PdfXObject* m_pXObj;

};

```

```
////////////////s
using namespace PoDoFo;
//#define CONVERSION_CONSTANT 0.002834645669291339 
///Conversation constant to convert 1/1000th mm to 1/72 inchs

#define  NUMPAT     64

TilePattern::TilePattern() :
	pPage(nullptr),
	m_pPainter(nullptr)
{
	pPage = doc.CreatePage(PdfPage::CreateStandardPageSize(ePdfPageSize_A4));
	m_pPainter = new PdfPainter;
}

TilePattern::~TilePattern()
{

}

void TilePattern::TestTilePattern()
{
	////////////////////////////////////////
	PdfPainter* pPainter = m_pPainter;
	if (pPainter == nullptr) return;
	pPainter->SetPage(pPage);

	/// fill back color
	FillBkGround();
	/// draw polyline with different style 
	drawPolyline();

	/////////////////////////////
	/// draw pattern
	if (1) {
		int patWid, patHei;
		patWid = patHei = NUMPAT;
		m_pXObj = createPattern(patWid, patHei, &doc, nullptr);

		bool bUseTilingPattern = true;
		if (bUseTilingPattern) {
			pPainter->Save();
			drawWithTilingPattern(&doc, m_pXObj, pPainter);
			pPainter->Restore();
		}
		else
		{
			drawWithXobject(m_pXObj, pPainter);
		}

		if (m_pXObj) { delete m_pXObj; }
	}///
	pPainter->FinishPage(); /// match with setPage

							/////////////////////////////
							//// save pdf document
	char szFile[64] = { "\0" };
	sprintf(szFile, "F:/cache/%d_tilePattern.pdf", GetTickCount());
	Save(szFile);
}


void TilePattern::initPattern(int patWid, int patHei, PdfXObject* pXObj, PdfDocument* pParent)
{
	PdfPainter patPainter;

	/// construct pattern

	patPainter.SetPage(pXObj); /// set XObject as the render target

								/// fill back color
	{
		double r = 0.6;
		double g = 0.6;
		double b = 0.0;
		patPainter.SetColor(r, g, b);
		patPainter.Rectangle(0, 0, patWid, patHei);
		patPainter.Fill();
	}

	/// draw line
	const double dsize = 7.99; ///pattern base size is 8.0;
	if (1) {
		patPainter.SetStrokeWidth(50 * CONVERSION_CONSTANT);
		patPainter.SetLineCapStyle(ePdfLineCapStyle_Round);
		patPainter.SetLineJoinStyle(ePdfLineJoinStyle_Miter);
		patPainter.SetStrokingColor(1.0, 0.0, 0.0);

		patPainter.MoveTo(0.0, 0.0);
		patPainter.LineTo(dsize, dsize);
		patPainter.MoveTo(0.0, dsize);
		patPainter.LineTo(dsize, 0.0);
		patPainter.Stroke();
	}
	/// draw polyline
	{
		double xbase = 0;
		double ybase = 0;
		double offset = dsize;
		int ncount = 5;
		D_Point dots[5] = { xbase ,ybase,xbase + offset,ybase,xbase + offset,ybase + offset,xbase,ybase + offset,xbase + offset*0.99,ybase + offset*0.01 };

		patPainter.SetStrokingColor(0.0, 0.0, 1.0);
		i_AddPath(&patPainter, dots, ncount);
		patPainter.SetStrokeWidth(10 * CONVERSION_CONSTANT);
		patPainter.SetLineCapStyle(ePdfLineCapStyle_Round);
		patPainter.SetLineJoinStyle(ePdfLineJoinStyle_Miter);
		patPainter.Stroke();
	}
	/// draw text
	{
		PdfFont* pFont = pParent->CreateFont("Comic Sans MS");
		pFont->SetFontSize(1.0);
		patPainter.SetFont(pFont);
		patPainter.SetColor(0.0, 0.0, 1.0);
		patPainter.DrawText(1, 1000 * CONVERSION_CONSTANT, "Tile Pattern");
	}

	patPainter.FinishPage(); /// match with setPage
}

PdfXObject* TilePattern::createPattern(int patWid, int patHei, PdfDocument* pParent, PdfPainter* pPainter)
{
	PdfXObject* pXObj = new PdfXObject(PdfRect(0, 0, patWid, patHei), pParent);
	if (pPainter) {
		pPainter->SetPage(pXObj);
		double r = 1.0, g = 0.5, b = 0.0;
		pPainter->SetColor(r, g, b);
		pPainter->Rectangle(0, 0, patWid, patHei);
		pPainter->Fill();
		pPainter->FinishPage();
	}
	else
		initPattern(patWid, patHei, pXObj, pParent);
	return pXObj;
}

void TilePattern::SetMatrix(PdfTilingPattern& rPattern, double xscale, double yscale, double xoffset, double yoffset)
{
	PdfArray array;
	array.push_back(/*static_cast<pdf_int64>*/(xscale));
	array.push_back(static_cast<pdf_int64>(0));
	array.push_back(static_cast<pdf_int64>(0));
	array.push_back(/*static_cast<pdf_int64>*/(yscale));
	array.push_back(xoffset);
	array.push_back(yoffset);
	rPattern.GetObject()->GetDictionary().AddKey(PdfName("Matrix"), array);
	//rPattern.GetObject()->GetDictionary().AddKey(PdfName("XStep"), static_cast<int64_t>(10));
	//rPattern.GetObject()->GetDictionary().AddKey(PdfName("YStep"), static_cast<int64_t>(10));
}

void TilePattern::drawWithTilingPattern(PdfDocument* pParent, PdfXObject* pXObj, PdfPainter* pPainter)
{
	///pPainter->Save();
	double strokeR = 0.1, strokeG = 0.0, strokeB = 1.0;
	bool doFill = false; double fillR = 0.7, fillG = 0.7, fillB = 0.7;
	double offsetX = NUMPAT, offsetY = NUMPAT;
	if (0)
	{
		if (!pParent)  return;
		PdfImage pdfImage(pParent);
		EPdfTilingPatternType eTilingType = ePdfTilingPatternType_Image;
		pdfImage.LoadFromFile("F:/User/Pictures/icon-29-2x.png");
		double h = pdfImage.GetHeight();
		double w = pdfImage.GetWidth();
		offsetX = w * 3, offsetY = h * 3;
		PdfTilingPattern rPattern(eTilingType, strokeR, strokeG, strokeB, doFill, fillR, fillG, fillB,
			offsetX, offsetY, &pdfImage, pParent);
		pPainter->SetTilingPattern(rPattern);
	}
	else
	{
		PdfObject* pContent = pXObj->GetObject();
		PdfVecObjects* objs = pContent->GetOwner();

		PdfStream* pstream = pContent->GetStream();
		char* pBuffer = nullptr; pdf_long  lLen = 0;
		pstream->GetCopy(&pBuffer, &lLen);
		pstream->GetFilteredCopy(&pBuffer, &lLen);

		PdfRect rct = pXObj->GetPageSize();
		EPdfTilingPatternType eTilingType = ePdfTilingPatternType_DiagCross;
		offsetX = 0;
		offsetY = 0;
		double dmi = 72 / 25.4;
		PdfTilingPattern rPattern(eTilingType, strokeR, strokeG, strokeB, doFill, fillR, fillG, fillB,
			offsetX, offsetY, nullptr, pContent->GetOwner());

		double basesize = 8.0;
		double xscale = rct.GetWidth() / basesize;
		double yscale = rct.GetHeight() / basesize;
		double scale = 3.9;

		SetMatrix(rPattern, xscale, yscale, offsetX, offsetY);

		TVecFilters vecFlate;
		vecFlate.push_back(ePdfFilter_FlateDecode);

		if (0) {
			std::ostringstream out; fillB = 1.0;
			out << fillR << " " << fillG << " " << fillB << " rg" << " ";
			out << 0 << " " << 0 << " " << 30 << " " << 20 << " re" << " ";
			out << "f" << " ";
			std::string str = out.str();
			PdfMemoryInputStream stream(str.c_str(), str.length());
			rPattern.GetObject()->GetStream()->Set(&stream, vecFlate);
		}
		else {
			PdfMemoryInputStream stream(pBuffer, lLen);
			rPattern.GetObject()->GetStream()->Set(&stream, vecFlate);
		}

		pPainter->SetTilingPattern(rPattern);
	}

	/// fill the tiling pattern
	{
		const double     x = 10000 * CONVERSION_CONSTANT;
		const double     y = 260000 * CONVERSION_CONSTANT; // 27cm
		const double   dWidth = 180000 * CONVERSION_CONSTANT; // 18cm
		size_t size = floor(dWidth / NUMPAT);
		double ybase = y - NUMPAT*(size - 2) / 2;
		double xmid = (x + NUMPAT*(size - 2) / 2 + x)*0.5;
		int ncount = 4;
		D_Point dots[4] = { x,ybase,NUMPAT*(size - 1) + x,ybase,xmid,y - dWidth*0.8,x + (x + xmid)*0.1,(ybase + y - dWidth)*0.5 };
		i_AddPath(pPainter, dots, ncount);

		pPainter->ClosePath();
		pPainter->Fill();

		//	pPainter->Clip(true);
	}
	//pPainter->Restore();
}

void TilePattern::drawPolyline()
{
	PdfPainter* pPainter = m_pPainter;
	if (pPainter == nullptr) return;
	double xbase = NUMPAT;
	double ybase = NUMPAT;
	double xmid = 0.5*NUMPAT;
	int ncount = 4;
	D_Point dots[4] = { xbase ,ybase,xbase + xmid,ybase,xbase + xmid,ybase + xmid,xbase,ybase + xmid };

	pPainter->SetStrokingColor(0.0, 0.0, 1.0);
	i_AddPath(pPainter, dots, ncount);
	pPainter->SetStrokeWidth(1000 * CONVERSION_CONSTANT);
	pPainter->SetLineCapStyle(ePdfLineCapStyle_Round);
	pPainter->SetLineJoinStyle(ePdfLineJoinStyle_Miter);
	pPainter->Stroke();

	pPainter->SetStrokingColor(1.0, 0.0, 0.0);
	i_AddPath(pPainter, dots, ncount);
	pPainter->SetStrokeWidth(500 * CONVERSION_CONSTANT);
	pPainter->SetLineCapStyle(ePdfLineCapStyle_Square);
	pPainter->SetLineJoinStyle(ePdfLineJoinStyle_Bevel);
	pPainter->Stroke();
}

void TilePattern::FillBkGround()
{
	double     x = 10000 * CONVERSION_CONSTANT;
	double     y = pPage->GetPageSize().GetHeight() - 10000 * CONVERSION_CONSTANT;
	const double dWidth = 180000 * CONVERSION_CONSTANT; // 18cm
	const double dHeight = 270000 * CONVERSION_CONSTANT; // 27cm
	size_t size = floor(dWidth / NUMPAT);

	PdfPainter* pPainter = m_pPainter;
	if (pPainter == nullptr) return;
	pPainter->SetColor(0.6, 0.6, 0.6);
	pPainter->Rectangle(x, y - dHeight, dWidth, dHeight);
	pPainter->Fill();
}

int TilePattern::i_AddPath(PdfPainter* painter, D_Point* dots, int count)
{
	if (!painter || dots == NULL || count < 2)
	{
		return 0;
	}
	painter->MoveTo(dots[0].x, dots[0].y);
	for (int i = 1; i < count; ++i)
	{
		painter->LineTo(dots[i].x, dots[i].y);
	}
	return 1;
}

void TilePattern::drawWithXobject(PdfXObject* pXObj, PdfPainter* pPainter)
{
	const double     x = 10000 * CONVERSION_CONSTANT;
	const double     y = 260000 * CONVERSION_CONSTANT; // 27cm
	const double   dWidth = 180000 * CONVERSION_CONSTANT; // 18cm
	size_t size = floor(dWidth / NUMPAT);
	double ybase = y - NUMPAT*(size - 2) / 2;
	double xmid = (x + NUMPAT*(size - 2) / 2 + x)*0.5;
	int ncount = 4;
	D_Point dots[4] = { x,ybase,NUMPAT*(size - 1) + x,ybase,xmid,y - dWidth*0.8,x + (x + xmid)*0.1,(ybase + y - dWidth)*0.5 };
	i_AddPath(pPainter, dots, ncount);

	pPainter->ClosePath();
	//pPainter->Save();
	pPainter->SetGray(0.6);
	pPainter->Fill();
	//pPainter->Restore();
	//pPainter->Clip(true);
	for (size_t index = 1; index < size - 1; index++)
	{
		pPainter->DrawXObject(NUMPAT*index + x, y - NUMPAT*index, pXObj);
	}
}

void TilePattern::Save(const char* pszFile)
{
	doc.Write(pszFile);
}

```

# 112 encrypt and decrypt in java

```
  /**
	* 加密原始密码
	*
	* @param password 原密码
	* @param key   密钥  /使用DES  密钥必须是8个字节  //使用AES  密钥必须是16个字节s
	* @return 加密后的密码
	*/
public static String encrypt(String password, String key) {
	try {
		SecretKey secretKey = SecretKeyFactory.getInstance("des").generateSecret(new DESKeySpec(key.getBytes()));
		Cipher cipher = Cipher.getInstance("des");
		cipher.init(Cipher.ENCRYPT_MODE, secretKey, new SecureRandom());
		byte[] cipherData = cipher.doFinal(password.getBytes());
		return DatatypeConverter.printBase64Binary(cipherData);
	} catch (Exception ex) {
		return password;
	}
}

  /**
	* 解密密码
	*
	* @param encryptedPassword 加密后的密码
	* @param key            密钥
	* @return 原密码
	*/
public static String decrypt(String encryptedPassword, String key) {
	try {
		SecretKey secretKey = SecretKeyFactory.getInstance("des").generateSecret(new DESKeySpec(key.getBytes()));
		Cipher cipher = Cipher.getInstance("des");
		cipher.init(Cipher.DECRYPT_MODE, secretKey, new SecureRandom());
		byte[] plainData = cipher.doFinal(DatatypeConverter.parseBase64Binary(encryptedPassword));
		return new String(plainData);
	} catch (Exception ex) {
		return encryptedPassword;
	}
}

```

# 113  create guid

```
namespace cycle
{
int sprintf_s(char *_Dest, size_t maxlen, const char * _Format, ...)
{
	int n = 0;
	va_list ap;
	va_start(ap, _Format);
	n = vsnprintf(_Dest, maxlen, _Format, ap);  
	//vsnprintf才是接收va_list参数的版本，原逻辑误用成snprintf导致传入的值有问题
	va_end(ap);
	return n;
}


int swprintf_w(LPWSTR pszOut, LPCWSTR format, ...)
{
	int n = 0;
	va_list ap;
	va_start(ap, (const wchar_t *)format);
	n = vswprintf((wchar_t *)pszOut, 1024, (const wchar_t *)format, ap);
	va_end(ap);
	return(n);
}

#ifdef _use_uuid_qt
#include <QtCore/QUuid>
#else
#include <uuid/uuid.h>
#endif

HRESULT  CreateGuid(GUID  * pguid)
{
#ifdef _use_uuid_qt
    QUuid uuid = QUuid::createUuid();
	if(pguid)
	{
		pguid->Data1 = uuid.data1;
		pguid->Data2 = uuid.data2;
		pguid->Data3 = uuid.data3;
		for (int i = 0; i < 8; ++i)
			pguid->Data4[i] = uuid.data4[i];
	}
#else
	uuid_t uu;
	uuid_generate(uu);
	if (pguid)
		memcpy(pguid, &uu, sizeof(uuid_t));
#endif	
	return 1;
}


}

#ifdef CYCLE_UNICODE

typedef WCHAR TCHAR__;
#define CLESTR(str) L##str

#else  /* CYCLE_UNICODE */

typedef char TCHAR__;
#define CLESTR(str) str

#endif /* CYCLE_UNICODE */

typedef TCHAR__ CLECHAR, *LPCLECHAR, *LPCLESTR;

int StringFromGUID2(const GUID& rguid, LPCLESTR lpsz, int cchMax)
{
	char buffer[64] = { 0 };
	GUID guid = rguid;
	cycle::sprintf_s(buffer,cchMax,
		"%08X-%04X-%04x-%02X%02X-%02X%02X%02X%02X%02X%02X",
		guid.Data1, guid.Data2, guid.Data3,
		guid.Data4[0], guid.Data4[1], guid.Data4[2],
		guid.Data4[3], guid.Data4[4], guid.Data4[5],
		guid.Data4[6], guid.Data4[7]);
	
	if(lpsz)
		memcpy(lpsz, buffer, strlen(buffer));
	return 1;
}

void runGUID()
{
		GUID            g_guid;
		CLECHAR         str1[50];
		 
		cycle::CreateGuid(&g_guid);

		USES_CONVERSION;
		memset(str1,0,sizeof(CLECHAR) * 50);
		StringFromGUID2(g_guid,str1,50);
}


```


# 114  a2wconverter and  w2aconverter

```
class converter_Content_
{
public:
	CLECHAR *a2wconverter(const char *pszSrc)
	{
		stringx str(pszSrc);
		m_wstring = str.ToStdU16String();
		return (CLECHAR *)m_wstring.c_str();
	}
	char *w2aconverter(const CLECHAR *wcharStr)
	{
		std::u16string srcStr = (char16_t*)wcharStr;
		stringx str = stringx::FromStdU16String(srcStr,stringx::GB18030);
		m_string = str.StdString();
		return (char*)m_string.c_str();
	}
public:
	converter_Content_()
	{}
	~converter_Content_(){}
private:
	std::u16string  m_wstring;
	std::string   m_string;
};


#define USES_CONVERSION	converter_Content_  global_convert;

#define W2A(a)	        global_convert.w2aconverter((const CLECHAR *)a)
#define A2W(w)	        global_convert.a2wconverter(w)


```


# 115  GetSymmetryDot

```
 /// 求一点的中心对称点
///
/// @param [in] dot 点
/// @param [in] centerDot 中心点
/// @param [out] MiirrorDot 中心对称后的点
///
/// @return 1 成功；0 失败
long  GetSymmetryDot(DOT dot,DOT centerDot,DOT* MiirrorDot)
{
	if(MiirrorDot)
	{
		MiirrorDot->x = 2*centerDot.x - dot.x;
		MiirrorDot->y = 2*centerDot.y - dot.y;
	}
	return 1;
}

/// 计算两点之间的弧度(0~2*PI)之间 
///
/// @param [in] x1 点1x坐标
/// @param [in] y1 点1y坐标
/// @param [in] x2 点2x坐标
/// @param [in] y2 点2y坐标
/// @param [out] ang2 弧度
void ComputeAngle(double x1,double y1, double x2,double y2,double *ang2)
{
    if(fabs(x2-x1) <= EPS)
	{
		*ang2=PI/2.0;
		if(y2-y1<0.)
			*ang2+=PI;
	}
    else
	{
		*ang2 = atan((y2-y1)/(x2-x1));
		if(x2-x1<0.)
			*ang2+=PI;
	}
    if(*ang2<0.)
		*ang2+=2.0*PI;
    return;
}

//================================RotateDot()===============================
/// 求点旋转后的位置
///
/// @param [in, out] dotX 需要旋转的点坐标：x
/// @param [in, out] dotY 需要旋转的点坐标：y
/// @param [in] centerX 旋转中心坐标：x
/// @param [in] centerY 旋转中心坐标：y
/// @param [in] dAngle 逆时针旋转角度（弧度单位）
///
/// @return true
//================================================================================
template<class _T> inline
bool RotateDot(_T &dotX,_T &dotY,_T centerX,_T centerY,double dAngle)
{
	_T		x,y;
	x = dotX - centerX;
	y = dotY - centerY;
	dotX = centerX + x*cos(dAngle) - y*sin(dAngle);
	dotY = centerY + x*sin(dAngle) + y*cos(dAngle);
	return true;
}

//==================================ZoomDot()=================================
/// 求点缩放后的位置
///
/// @param [in] dotX 点x坐标
/// @param [in] dotY 点y坐标
/// @param [in] centerX 缩放中心x坐标
/// @param [in] centerY 缩放中心y坐标
/// @param [in] dScaleX x方向缩放系数
/// @param [in] dScaleY y方向缩放系数
///
/// @return true
//================================================================================
template<class _T> inline
bool ZoomDot(_T &dotX,_T &dotY,_T centerX,_T centerY,double dScaleX,double dScaleY)
{
	_T		x,y;
	x = dotX - centerX;
	y = dotY - centerY;
	dotX = centerX + x*dScaleX;
	dotY = centerY + y*dScaleY;
	return true;
}

```


# 116  DISPLAY=localhost

```

Exception in thread "main" java.lang.reflect.InvocationTargetException
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at sun.launcher.LauncherHelper$FXHelper.main(LauncherHelper.java:767)
Caused by: java.lang.UnsupportedOperationException: Unable to open DISPLAY
	at com.sun.glass.ui.gtk.GtkApplication.<init>(GtkApplication.java:68)
	at com.sun.glass.ui.gtk.GtkPlatformFactory.createApplication(GtkPlatformFactory.java:41)
	at com.sun.glass.ui.Application.run(Application.java:146)
	at com.sun.javafx.tk.quantum.QuantumToolkit.startup(QuantumToolkit.java:257)
	at com.sun.javafx.application.PlatformImpl.startup(PlatformImpl.java:211)
	at com.sun.javafx.application.LauncherImpl.startToolkit(LauncherImpl.java:675)
	at com.sun.javafx.application.LauncherImpl.launchApplicationWithArgs(LauncherImpl.java:337)
	at com.sun.javafx.application.LauncherImpl.launchApplication(LauncherImpl.java:328)
	... 5 more
root@user100:/opt/apps/program# echo $DISPLAY
:10.0
root@user100:/opt/apps/program# export DISPLAY=localhost:10.0

```


# 117 set solib-search-path in GDB Session


##  case
when attached the target linux host via visualGDB tool,
we need set the library path in order to debug the target library, 
usually we send  this command  "set solib-search-path %LIB_path" in GDB Session;
eg: set solib-search-path /opt/apps/program

but sometimes the next error message crash us;

```
VisualGDB is licensed to Company (site license)
gdb --interpreter mi
All created breakpoints are pending. Setting a breakpoint at main()...
Cannot execute command. GDB is not ready.
Please halt the debugged program by selecting Debug->Break All.
```

##  solution
Base on the message above : we need selecting Debug->Break All in Visual studio;

1. after selecting Debug->Break All  (CTRL+ATL+BREAK),
2. send the command again in GDB Session;
then we get the following configure information;

```

set solib-search-path /opt/apps/program
&"set solib-search-path /opt/apps/program\n"
Reading symbols from /opt/apps/program/libz.so...
(No debugging symbols found in /opt/apps/program/libz.so)
Reading symbols from /opt/apps/program/libpng16.so.16...
(No debugging symbols found in /opt/apps/program/libpng16.so.16)
Reading symbols from /opt/apps/program/libpcre.so.3...
(No debugging symbols found in /opt/apps/program/libpcre.so.3)
Reading symbols from /opt/apps/program/libvtkCommonSystem-8.2.so.1...
Reading symbols from /opt/apps/program/libvtkCommonTransforms-8.2.so.1...
Reading symbols from /opt/apps/program/libvtkDICOMParser-8.2.so.1...
Reading symbols from /opt/apps/program/libvtkmetaio-8.2.so.1...
OK
```

then we can debug the target library;



# 118  remote debug in IDEA


how to  debug remote program which run in linux environment with  IDEA?

## solution

1. prepare and install IDEA ,and configure jdk ;

2. in IDEA, open jar module,compile and package it into  target jar file "target.jar"; 
 copy the atrget jar file into target host which u wanna debug;

3. in IDEA Run/Debug Configurations,add new Configuration by selecting 'Romote'  ;then configure 'Remote' ;
  ```
  Debugger mode: Attach to remote JVM
  Transport : Socket
  Host: 192.168.23.112
  Port: 50505
  Commmand line arguments for remote JVM：
  -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=50505
  use module classpath: target.module
  ```
  then apply this configuration; NOTE: Port  is any free number;

4. next is the key;
   open target linux host which has ip list above (eg:192.168.23.112 );
   in this host ,append the original start script with  the command line arguments which list in step 3:

   eg:
   ```
   原来远程启动脚本
   java -jar xxx.jar
   调整后的调试启动脚本
   java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005  -jar xxx.jar

    -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005
    根据idea配置里远程jvm的命令行参数生成。
   ```
   then run the adjusted start script ; process the target fucntions;
5. in IDEA, click the Remote debug;

6. ok,next we can debug the remote program

# 119  caculate math

 ```
//计算直线角度（-PI到+PI）
double   Caculate_PIAngle(double x0,double y0,double x1,double y1)
{
	double ang;
	double  dx,dy;
	//       ^y
	dy=y1-y0;                   //       |
	dx=x1-x0;                   //      PI/2
	if(dy||dx)                  //       |
		ang=atan2(dy,dx);  //----//-(+PI)-|---0--->x
	else			//  -PI  |
		ang=0;                   //       |
	return(ang);                //       -PI/2
}

//计算直线角度（0到2PI）

double   Caculate_2PIAngle(double x0,double y0,double x1,double y1)
{
	double ang;
	ang=Caculate_PIAngle(x0,y0,x1,y1);
	if(ang<0)ang+=2*PI;
	return(ang);
}


//计算向量角度
double WINAPI Caculate_PIAngleOfArr1(double dx,double dy)
{
	double ang;
								//       ^y
								//       |
								//      PI/2
	if(dy||dx)                   //       |
		ang=atan2(dy,dx);    //- //-(+PI)-|---0--->x
	else			             // -PI   |
		ang=0;                   //       |
	return(ang);                //       -PI/2
}



//计算线段长度

double   _SegmentLength1(double x0,double y0,double x1,double y1)
{
	double dx,dy;
	dx=x1-x0;
	dy=y1-y0;
	return(sqrt((double)dx*dx+(double)dy*dy));
}

double   CalculateLength(void *xyz,long len,short dim)
{

	long  i;
	double ds=0;
	if(dim==3)
	{
		D_3DOT  *p0;
		D_3DOT  *p1;

		p0=(D_3DOT  *)xyz;
		p1=p0+1;
		for(i=1;i<len;i++)
		{
			ds+=sqrt((p1->x-p0->x)*(p1->x-p0->x)+(p1->y-p0->y)*(p1->y-p0->y)+(p1->z-p0->z)*(p1->z-p0->z));
			p0++;
			p1++;
		}

	}
	else
	{
		D_DOT  *p0;
		D_DOT  *p1;

		p0=(D_DOT *)xyz;
		p1=p0+1;
		for(i=1;i<len;i++)
		{
			ds+=sqrt((p1->x-p0->x)*(p1->x-p0->x)+(p1->y-p0->y)*(p1->y-p0->y));
			p0++;
			p1++;
		}
	}
	return(ds);
}

 ```

# 120  setting maven local Repository

 Maven_PATH/.m2/settings
 ```
 <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                          https://maven.apache.org/xsd/settings-1.0.0.xsd">
	<localRepository>E:\usr\home\.m2\repository</localRepository>
    <mirrors>
        <mirror>
            <id>alimaven</id>
            <name>aliyun maven</name>
            <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
            <mirrorOf>central</mirrorOf>
        </mirror>
    </mirrors>
 ```
 
# 121  report the undefined symbol in dynamic library

 ```
[root@bg01 program]# ldd -r libdisplay.so
        linux-vdso.so.1 (0x0000fffd7bda0000)
        libXext.so.6 => /lib64/libXext.so.6 (0x0000fffd644a0000)
        libXrender.so.1 => /lib64/libXrender.so.1 (0x0000fffd64470000)
        libGLX.so.0 => /lib64/libGLX.so.0 (0x0000fffd64410000)
        libGLdispatch.so.0 => /lib64/libGLdispatch.so.0 (0x0000fffd642b0000)
        libxml2.so.2 => /lib64/libxml2.so.2 (0x0000fffd64120000)
undefined symbol: PntSymbolToPS      (./libdisplay.so)
undefined symbol: LinSymbolToPS      (./libdisplay.so)
undefined symbol: RegSymbolToPS      (./libdisplay.so)
 
 ```
--version：打印指令版本号；
-v：详细信息模式，打印所有相关信息；
-u：打印未使用的直接依赖；
-d：执行重定位和报告任何丢失的对象；
-r：执行数据对象和函数的重定位，并且报告任何丢失的对象和函数；
--help：显示帮助信息。
 
 # 122  how to use Log4j and set Log4J properties
 
 ## how to use Log4j
 
 ```
import lombok.extern.log4j.Log4j;

import java.io.File;
import java.nio.file.Paths;
import java.util.*;

///Slf4j
@Log4j
public class MapPrint {

  
    private void logi(String str) {
        curTime = System.currentTimeMillis();
        long totalsecond = curTime / 1000;

        long millsecond = curTime % 1000;
        long second = totalsecond % 60;

        long totalminute = totalsecond / 60;
        long minute = totalminute % 60;

        long totalhour = totalminute / 60;
        long hour = totalhour % 24;
        long delta = curTime - preTime; /// cost time
        String sta = String.format("%2d:%2d:%2d.%d cost:%6d ms |", hour + 8, minute, second, millsecond, delta);
        String strInfo = sta + str;

        log.info(strInfo);
        preTime = curTime;
    }
	
	void export(String srcFile,String templateFile,String saveFile)
	{
	    logi("1. addlabels: begin");
        addLabels(labels, "index");
        logi("1. addlabels: end");

        logi("2. applayTemplate: begin");
        String strRes = applayTemplate(srcFile, templateFile, saveFile.toPath().toAbsolutePath().toString(), format);
        logi("2. applayTemplate: end");

        logi("0.export: end");
        return strRes;
    }
	
}
 
 ```
 
  ## set Log4J properties
    1.save next block into file log4j.properties;
	2.then move it into folder 'targetJar\src\main\resources\'
  ```
	# Root logger option
	log4j.rootLogger=INFO, stdout, FILE
	# Direct log messages to stdout
	log4j.appender.stdout=org.apache.log4j.ConsoleAppender
	log4j.appender.stdout.Target=System.out
	log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
	log4j.appender.stdout.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1} - %m%n

	# Direct log messages to file

	# Define the file appender
	log4j.appender.FILE=org.apache.log4j.FileAppender
	# Set the name of the file
	log4j.appender.FILE.File=/data/soft/log/log2023.log
	# Set the immediate flush to true (default)
	log4j.appender.FILE.ImmediateFlush=true
	# Set the threshold to debug mode
	log4j.appender.FILE.Threshold=debug
	# Set the append to false, overwrite
	log4j.appender.FILE.Append=false
	# Define the layout for file appender
	log4j.appender.FILE.layout=org.apache.log4j.PatternLayout
	log4j.appender.FILE.layout.conversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1} - %m%n
```
  
 # 123 how to upload file to remote machine  from local host with SecureCRT 
 
 ```

1.快捷键alt+p打开sftp窗口，输入想要下载的文件所在的目录，回车
lcd E:/download
2.输入要下载的文件
get /usr/hello.jar
回车之后，就能将服务器中指定的文件下载到本地指定的目录下。

```
 
 # 124 install local jar file into maven  repository with 'mvn install-file' 
 
 ```
	"D:\Program\JetBrains\IntelliJ_IDEA_2019\plugins\maven\lib\maven3\bin\mvn" install:install-file -DgroupId=com.cycle.zone.igs -DartifactId=cycle-igs-api -Dversion=10.6.2.10 -Dpackaging=jar -Dfile=cycle-igs-api-10.6.2.10.jar
	 
	maven添加本地包命令mvn install:install-file各个属性详解

	答：

	1.添加带有别名core的jar包，common-auth-0.0.1-SNAPSHOT-core.jar

	mvn install:install-file -Dfile=F:/common-auth/target/common-auth-0.0.1-SNAPSHOT-core.jar -DgroupId=com.cloud -DartifactId=common-auth -Dversion=0.0.1-SNAPSHOT -Dclassifier=core  -Dpackaging=jar

	2.添加没别名的jar包，common-auth-0.0.1-SNAPSHOT.jar

	mvn install:install-file -Dfile=F:/common-auth/target/common-auth-0.0.1-SNAPSHOT.jar -DgroupId=com.cloud -DartifactId=common-auth -Dversion=0.0.1-SNAPSHOT -Dpackaging=jar

	1比较2多了一个属性Dclassifier，用来申明包的别名，也就是拼接在-Dversion的值0.0.1-SNAPSHOT的后面的名称，例如core。执行上面两条命令行后，在com/cloud/common-auth/0.0.1-SNAPSHOT/目录下添加了两个包，分别为：common-auth-0.0.1-SNAPSHOT-core.jar和common-auth-0.0.1-SNAPSHOT.jar。
```

```
	各个属性解释

	<groupId>com.bx.cloud</groupId>
	<artifactId>bx-common</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>pom</packaging>

	-Dfile：包的本地真实地址
	-DgroupId：pom.xml中groupId
	-DartifactId：pom.xml中artifactId
	-Dversion：pom.xml中0.0.1-SNAPSHOT
	-Dpackaging：jar或war，包的后缀名
	-Dclassifier：兄弟包的别名，也就是-Dversion值后面的字符common-auth-0.0.1-SNAPSHOT-core.jar的-core
 ```

```
Microsoft Windows [版本 6.1.7601]
版权所有 (c) 2009 Microsoft Corporation。保留所有权利。

C:\Users\Administrator>"D:/Program Files/JetBrains/IntelliJ IDEA 2019.2.2/plugins/maven/lib/maven3/bin/mvn" install:install-file -Dfile="F:\JXBrowser\lib\jxbrowser-6.23.1.jar" -DgroupId=com.teamdev.jxbrowser -DartifactId=jxbrowser -Dversion=6.23.1 -Dpackaging=jar
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------< org.apache.maven:standalone-pom >-------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] --------------------------------[ pom ]---------------------------------
[INFO]
[INFO] --- maven-install-plugin:2.4:install-file (default-cli) @ standalone-pom
---
[INFO] Installing F:\JXBrowser\lib\jxbrowser-6.23.1.jar to F:\.m2\repository\com\teamdev\jxbrowser\jxbrowser\6.23.1\jxbrowser-6.23.1.jar
[INFO] Installing C:\Users\ADMINI~1\AppData\Local\Temp\mvninstall150256564299579
2242.pom to F:\.m2\repository\com\teamdev\jxbrowser\jxbrowser\6.23.1\jxbrowser-6.23.1.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.514 s
[INFO] Finished at: 2023-06-01T19:36:33+08:00
[INFO] ------------------------------------------------------------------------

C:\Users\Administrator>

```
 
 # 125 leaflet quick-start example
 
https://leafletjs.cn/examples/quick-start/

 ```
 <!DOCTYPE html>
<!--https://leafletjs.cn/examples/quick-start/-->
<html lang="en">
<head>
	<base target="_top">
	<meta charset="utf-8">
	<meta name="viewport" content="width=device-width, initial-scale=1">
	
	<title>Quick Start - Leaflet</title>
	<!--link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.3/dist/leaflet.css"/-->
	<link rel="stylesheet" href="./leaflet.css"/>
	  <!-- Make sure you put this AFTER Leaflet's CSS -->
    <!--script src="https://unpkg.com/leaflet@1.9.3/dist/leaflet.js"></script-->
    
    <script src="./leaflet.js"></script>

	<style>
		<!--html, body {
			height: 100%;
			margin: 0;
		}
		
		#map { height: 900px; width:1280px }-->
		
		body {
			padding: 0;
			margin: 0;
		}
		html, body, #map {
			height: 100%;
			width: 100vw;
		}

	</style>

	
</head>
<body>
	<div id="map"></div>
</body>


<!-- 
https://webst01.is.autonavi.com/appmaptile?style=6&x={x}&y={y}&z={z}
http://wprd04.is.autonavi.com/appmaptile?lang=zh_cn&size=1&style=7&x={x}&y={y}&z={z}
https://wprd01.is.autonavi.com/appmaptile?x={x}&y={y}&z={z}&lang=zh_cn&size=1&scl=2&style=8&ltype=11
https://tile.openstreetmap.org/{z}/{x}/{y}.png
https://tile.openstreetmap.bzh/br/{z}/{x}/{y}.png
http://rt0.map.gtimg.com/realtimerender?z={z}&x={x}&y={-y}&type=vector&style=0
-->
<script>
	var map = L.map('map').setView([30.50, 114.49], 13);

	L.tileLayer('https://webst01.is.autonavi.com/appmaptile?style=6&x={x}&y={y}&z={z}', {
		maxZoom: 18,
		attribution: '&copy; <a href="https://cyclezone.github.io/"> cyclezone</a>'
	}).addTo(map);

	var greenIcon = L.icon({
		iconUrl: 'dist/images/marker-icon.png',
		shadowUrl: 'dist/images/marker-shadow.png',

		iconSize:     [35, 60], // size of the icon
		shadowSize:   [32, 64], // size of the shadow
		iconAnchor:   [2, 56], // point of the icon which will correspond to marker's location
		shadowAnchor: [4, 62],  // the same for the shadow
		popupAnchor:  [-3, -76] // point from which the popup should open relative to the iconAnchor
	});

	var marker = L.marker([30.478, 114.469], {icon: greenIcon}).addTo(map);
	
	var circle = L.circle([30.487, 114.4801], {
		color: 'red',
		fillColor: '#f03',
		fillOpacity: 0.5,
		radius: 500
	}).addTo(map);
	
	var polygon = L.polygon([
		[30.492, 114.496],
		[30.493, 114.52],
		[30.455, 114.518],
		[30.456, 114.489]
	]).addTo(map);
	
	marker.bindPopup("<b>Hello world!</b><br>I am a popup.").openPopup();
	circle.bindPopup("I am a circle.");
	polygon.bindPopup("I am a polygon.");
	
	var popup = L.popup()
    .setLatLng([30.513, 114.49])
    .setContent("I am a standalone popup.")
    .openOn(map);
	
	var popup = L.popup();

	function onMapClick(e) {
		popup
			.setLatLng(e.latlng)
			.setContent("You clicked the map at " + e.latlng.toString())
			.openOn(map);
	}

	map.on('click', onMapClick);

</script>

</html> 
 
 ```
 

 # 126 import jar dependency

## from local file

```
<dependency>
	<groupId>com.teamdev.jxbrowser</groupId>  <!--自定义-->
	<artifactId>jxbrowser</artifactId>    <!--自定义-->
	<version>6.23.1</version> <!--自定义-->
	<scope>system</scope> <!--system，类似provided，需要显式提供依赖的jar以后，Maven就不会在Repository中查找它-->
	<systemPath>${basedir}/lib/jxbrowser-6.23.1.jar</systemPath> <!--项目根目录下的lib文件夹下-->
</dependency>
```

## from local maven repository 

1. open cmd window; run mvn install
```
"D:/Program Files/JetBrains/IntelliJ IDEA 2019.2.2/plugins/maven/lib/maven3/bin/mvn" install:install-file -Dfile="F:\JXBrowser\lib\jxbrowser-6.23.1.jar" -DgroupId=com.teamdev.jxbrowser -DartifactId=jxbrowser -Dversion=6.23.1 -Dpackaging=jar
```
we get information:

```
[INFO] Scanning for projects...
[INFO]
[INFO] ------------------< org.apache.maven:standalone-pom >-------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] --------------------------------[ pom ]---------------------------------
[INFO]
[INFO] --- maven-install-plugin:2.4:install-file (default-cli) @ standalone-pom
---
[INFO] Installing F:\JXBrowser\lib\jxbrowser-6.23.1.jar to F:\.m2\repository\com\teamdev\jxbrowser\jxbrowser\6.23.1\jxbrowser-6.23.1.jar
[INFO] Installing C:\Users\ADMINI~1\AppData\Local\Temp\mvninstall150256564299579
2242.pom to F:\.m2\repository\com\teamdev\jxbrowser\jxbrowser\6.23.1\jxbrowser-6.23.1.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  0.514 s
[INFO] Finished at: 2023-06-01T19:36:33+08:00
[INFO] ------------------------------------------------------------------------
```

2. then in pom.xml, append following
```
<dependency>
	<groupId>com.teamdev.jxbrowser</groupId>
	<artifactId>jxbrowser</artifactId>
	<version>6.23.1</version>
</dependency>
```
 
 # 127 remote desktop with aspia

for more information, please visit following  url :

```
 https://aspia.org
or
 https://github.com/dchapyshev/aspia
```

0.  install aspia_router;
    get aspia help information
```
PS D:\Pragram\Aspia\Router> ./aspia_router.exe --help
aspia_router [switch]
Available switches:
        --install       Install service
        --remove        Remove service
        --start Start service
        --stop  Stop service
        --create-config Creates a configuration
        --keygen        Generating public and private keys
        --help  Show help
PS D:\Pragram\Aspia\Router> ./aspia_router.exe --keygen
Private key: E017C465E48A5AF5ADE189C8EC5393F0BE68550575889A3209F137E281718B5F
Public key: DBACEB30E64C6FEBE412BA0BF82F6E29C9A3FA1FD3509F734CB233FCE3D9FE0F
```
1.  create default config file
cd "D:\Program\Aspia\Router"
aspia_router --create-config
```
PS D:\Program\Aspia\Router> ./aspia_router.exe --create-config
Creation of initial configuration started.
Settings file path: "C:\\ProgramData\\aspia\\router.json"
Settings file does not exist yet.
Public key directory path: "C:\\ProgramData\\aspia"
Public key directory already exists.
Public key file: "C:\\ProgramData\\aspia\\router.pub"
Public key does not exist yet.
Creating a user...
User has been created. Adding a user to the database...
User was successfully added to the database.
Generating encryption keys...
Private and public keys have been successfully generated.
Writing a public key to a file...
Configuration successfully created. Don't forget to change your password!
User name: admin
Password: admin
Public key file: "C:\\ProgramData\\aspia\\router.pub"
```
3.  Service/daemon starting
Windows:
net start aspia-router
Linux:
sudo systemctl enable aspia-router
sudo service aspia-router start
```
PS D:\Pragram\Aspia\Router> net start aspia-router
Aspia Router Service 服务正在启动 .
Aspia Router Service 服务已经启动成功。
```
4.  install aspia_host;
    
4.1. Enabling the router in the settings
	1. Go to settings (Aspia -> Settings… -> Router)
	2. Enable the use of the Router
	3. Write the address of your Router, eg:192.168.10.50
	4. Write your Router's public key,eg: A1A903E7B53455FBE09EF72ADB432166BF4951FF862CDE618362C45C35FFE01B
```
	Open public key file and copy the public key. It will come in handy for configuring the Relay and Hosts.
	Windows:
	C:\ProgramData\aspia\router.pub
	Linux:
	/etc/aspia/router.pub
```

4.2 Add new user
   1. Go to settings (Aspia -> Settings… -> Users)
   2. add new user,input the user name and password;
   3. Enable the user acount and sessions type

4.3 then we get information:
   connected to a router

5. install aspia client
   input the address and choose sessions;
   then connect it, input user and password;

6. here we go, we can connect the remote machine

7.if wanna stop the service,do next;
Service/daemon stopping
Windows:
net stop aspia-router
Linux:
sudo service aspia-router stop

```
PS D:\Pragram\Aspia\Router> net stop aspia-router
Aspia Router Service 服务正在停止.
Aspia Router Service 服务已成功停止。
```

# 128 compile curl in linux

1. copy curl project codes into folder '~/code/curl/'
2. unzip it if it is a zip file;
3. locate the project; get auto configure file;
4. configue it and make install 

```
cycle@zone23112:~/code/curl$ ls
curl-tiny-curl-7_79  curl-tiny-curl-7_79.zip
cycle@zone23112:~/code/curl$ cd curl-tiny-curl-7_79/
cycle@zone23112:~/code/curl/curl-tiny-curl-7_79$ ls
acinclude.m4   CHANGES         COPYING         include        MacOSX-Framework  packages  README.md      src                 zuul.d
appveyor.yml   CMake           curl-config.in  lib            Makefile.am       plan9     RELEASE-NOTES  tests
buildconf      CMakeLists.txt  docs            libcurl.pc.in  Makefile.dist     projects  scripts        TINY-RELEASE-NOTES
buildconf.bat  configure.ac    GIT-INFO        m4             maketgz           README    SECURITY.md    winbuild

cycle@zone23112:~/code/curl/curl-tiny-curl-7_79$ autoreconf -fi
libtoolize: putting auxiliary files in '.'.
libtoolize: copying file './ltmain.sh'
libtoolize: putting macros in AC_CONFIG_MACRO_DIRS, 'm4'.
libtoolize: copying file 'm4/libtool.m4'
libtoolize: copying file 'm4/ltoptions.m4'
libtoolize: copying file 'm4/ltsugar.m4'
libtoolize: copying file 'm4/ltversion.m4'
libtoolize: copying file 'm4/lt~obsolete.m4'
libtoolize: Remember to add 'LT_INIT' to configure.ac.
configure.ac:120: installing './compile'
configure.ac:293: installing './config.guess'
configure.ac:293: installing './config.sub'
configure.ac:125: installing './missing'
docs/examples/Makefile.am: installing './depcomp'
```


# 129 Found package configuration file,but it set CURL_FOUND to FALSE

Question:

 Found package configuration file:E:/CURL/CMake/curl-config.cmake
  but it set CURL_FOUND to FALSE so package "CURL" is considered to be NOTFOUND. 
  Reason given by package:
  Following required components not found: ;curl;libcurl

from file 'curl-config.cmake',we get following fragment:

```
if(NOT CURL_FIND_COMPONENTS)
    set(CURL_FIND_COMPONENTS curl libcurl)
    if(CURL_FIND_REQUIRED)
        set(CURL_FIND_REQUIRED_curl TRUE)
        set(CURL_FIND_REQUIRED_libcurl TRUE)
    endif()
endif()

set(_curl_missing_components)
foreach(_comp ${CURL_FIND_COMPONENTS})
    if(EXISTS "${_DIR}/${_comp}-target.cmake")
        include("${_DIR}/${_comp}-target.cmake")
        set(CURL_${_comp}_FOUND TRUE)
    else()
        set(CURL_${_comp}_FOUND FALSE)
        if(CURL_FIND_REQUIRED_${_comp})
            set(CURL_FOUND FALSE)
            list(APPEND _curl_missing_components ${_comp})
        endif()
    endif()
endforeach()
```

CURL_FIND_COMPONENTS has been set 'curl libcurl';
so 'foreach(_comp ${CURL_FIND_COMPONENTS}) ', _comp should be curl or libcurl;
if set(CURL_FOUND FALSE), that mean  "${_DIR}/${_comp}-target.cmake" is NOT EXIST;
so should find and put  'curl-target.cmake' or 'libcurl-target.cmake' into ${_DIR};


# 130  *-config.cmake

```
get_filename_component(_libxml2_rootdir ${CMAKE_CURRENT_LIST_DIR}/../../../ ABSOLUTE)

set(LIBXML2_VERSION_MAJOR  2)
set(LIBXML2_VERSION_MINOR  9)
set(LIBXML2_VERSION_MICRO  3)
set(LIBXML2_VERSION_STRING "2.9.3")
set(LIBXML2_INSTALL_PREFIX ${_libxml2_rootdir})
set(LIBXML2_INCLUDE_DIRS   ${_libxml2_rootdir}/include ${_libxml2_rootdir}/include/libxml2)
set(LIBXML2_LIBRARY_DIR    ${_libxml2_rootdir}/lib)
set(LIBXML2_LIBRARIES      -L${LIBXML2_LIBRARY_DIR} -lxml2)

include(CMakeFindDependencyMacro)
find_dependency(JPEG)
set(libyuv_INCLUDE_DIRS "${CMAKE_CURRENT_LIST_DIR}/../../include")
include("${CMAKE_CURRENT_LIST_DIR}/libyuv-targets.cmake")

# Read in the exported definition of the library
include ("${_DIR}/proj4-targets.cmake")
include ("${_DIR}/proj4-namespace-targets.cmake")

```


# 131  *-targets.cmake

```
# Generated by CMake

if("${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION}" LESS 2.5)
   message(FATAL_ERROR "CMake >= 2.6.0 required")
endif()
cmake_policy(PUSH)
cmake_policy(VERSION 2.6)
#----------------------------------------------------------------
# Generated CMake target import file.
#----------------------------------------------------------------

# Commands may need to know the format version.
set(CMAKE_IMPORT_FILE_VERSION 1)

# Protect against multiple inclusion, which would fail when already imported targets are added once more.
set(_targetsDefined)
set(_targetsNotDefined)
set(_expectedTargets)
foreach(_expectedTarget libyuv)
  list(APPEND _expectedTargets ${_expectedTarget})
  if(NOT TARGET ${_expectedTarget})
    list(APPEND _targetsNotDefined ${_expectedTarget})
  endif()
  if(TARGET ${_expectedTarget})
    list(APPEND _targetsDefined ${_expectedTarget})
  endif()
endforeach()
if("${_targetsDefined}" STREQUAL "${_expectedTargets}")
  unset(_targetsDefined)
  unset(_targetsNotDefined)
  unset(_expectedTargets)
  set(CMAKE_IMPORT_FILE_VERSION)
  cmake_policy(POP)
  return()
endif()
if(NOT "${_targetsDefined}" STREQUAL "")
  message(FATAL_ERROR "Some (but not all) targets in this export set were already defined.\nTargets Defined: ${_targetsDefined}\nTargets not yet defined: ${_targetsNotDefined}\n")
endif()
unset(_targetsDefined)
unset(_targetsNotDefined)
unset(_expectedTargets)


# Compute the installation prefix relative to this file.
get_filename_component(_IMPORT_PREFIX "${CMAKE_CURRENT_LIST_FILE}" PATH)
get_filename_component(_IMPORT_PREFIX "${_IMPORT_PREFIX}" PATH)
if(_IMPORT_PREFIX STREQUAL "/")
  set(_IMPORT_PREFIX "")
endif()

# Create imported target libyuv
add_library(libyuv STATIC IMPORTED)

set_target_properties(libyuv PROPERTIES
  INTERFACE_COMPILE_DEFINITIONS "\$<\$<CONFIG:Debug>:YUV_DEBUG>;\$<\$<BOOL:>:YUV_IMPORT>"
  INTERFACE_INCLUDE_DIRECTORIES "${_IMPORT_PREFIX}/include"
)

# Create imported target GEOS::geos_c
#add_library(GEOS::geos_c SHARED IMPORTED)
#set_target_properties(GEOS::geos_c PROPERTIES
#  INTERFACE_INCLUDE_DIRECTORIES "${_IMPORT_PREFIX}/include/geos;${_IMPORT_PREFIX}/include"
#)

if(CMAKE_VERSION VERSION_LESS 3.0.0)
  message(FATAL_ERROR "This file relies on consumers using CMake 3.0.0 or greater.")
endif()
# Load information for each installed configuration.
get_filename_component(_DIR "${CMAKE_CURRENT_LIST_FILE}" PATH)
file(GLOB CONFIG_FILES "${_DIR}/libyuv-targets-*.cmake")
foreach(f ${CONFIG_FILES})
  include(${f})
endforeach()

# Cleanup temporary variables.
set(_IMPORT_PREFIX)

# Loop over all imported files and verify that they actually exist
foreach(target ${_IMPORT_CHECK_TARGETS} )
  foreach(file ${_IMPORT_CHECK_FILES_FOR_${target}} )
    if(NOT EXISTS "${file}" )
      message(FATAL_ERROR "The imported target \"${target}\" references the file
   \"${file}\"
but this file does not exist.  Possible reasons include:
* The file was deleted, renamed, or moved to another location.
* An install or uninstall procedure did not complete successfully.
* The installation package was faulty and contained
   \"${CMAKE_CURRENT_LIST_FILE}\"
but not all the files it references.
")
    endif()
  endforeach()
  unset(_IMPORT_CHECK_FILES_FOR_${target})
endforeach()
unset(_IMPORT_CHECK_TARGETS)

# This file does not depend on other imported targets which have
# been exported from the same project but in a separate export set.

# Commands beyond this point should not need to know the version.
set(CMAKE_IMPORT_FILE_VERSION)
cmake_policy(POP)
```

# 132  set vscode cache folder

D:\VSCode-win32-x64-1.29.1\Code.exe --user-data-dir "F:\Cache\vscode"

# 133 clear cache

C:\Users\Administrator\AppData\Local\Microsoft\VisualStudio\14.0\Extensions\qj0yglbx.blu\Data\vs14_1\Proj*
F:\Cache\vscode\User\workspaceStorage\3f96f295c1cfc7b0add796d6f79b3960\ms-vscode.cpptools\*

# 134  OpenMP

在OpenMP中，锁的数据类型为omp_lock_t，其基本功能就是保证线程同步。以下为常见的几个函数：

函数名	                                 描述
void omp_init_lock(omp_lock_t *);	    初始化线程锁
void omp_set_lock(omp_lock_t *);	    获得线程锁
void omp_nuset_lock(omp_lock_t *);	    释放线程锁
void omp_destroy_lock(omp_lock_t *);	销毁线程锁

```
#include<stdio.h>
#include<stdlib.h>
#include<omp.h>

int main(int argc, char *argv[])
{
    int x = 3, y = 4;
   
    // 初始化锁
    omp_lock_t lock;
    omp_init_lock(&lock);
    
    // 伪指令设置并行，使用num_threads()设置线程数，
    // 并且将x、y设置成shared状态
    // 花括号必须在下一行，而不能在#pragma后面
    #pragma omp parallel num_threads(3) shared(x, y)
    {
        omp_set_lock(&lock);  // 线程获得锁，保证同步
        
        x += omp_get_thread_num();  // 当前的线程id
        y += omp_get_thread_num();
        
        omp_unset_lock(&lock);  // 释放锁
    }
    
    printf("x = %d, y = %d\n", x, y);
    
    omp_destroy_lock(&lock);
    
    return 0;
}
```

```
#include<iostream>
#include<omp.h>
using namespace std;

void test_omp_for() {
    #pragma omp parallel for num_threads(3)
    for (int i = 0; i < 3; i++)
    {
        cout << "Hello, I am " << omp_get_thread_num() << ", iter " << i << endl;  // 当前的线程id
    }
}

int main(int argc, char* argv[])
{
    test_omp_for();
    return 0;
}
```


# 135 find_package

find_package的两种模式，一种是Module模式,另一种叫做Config模式，

在Module模式中，cmake需要找到一个叫做Find<LibraryName>.cmake的文件。
这个文件负责找到库所在的路径，为我们的项目引入头文件路径和库文件路径。
cmake搜索这个文件的路径有两个，一个是上文提到的cmake安装目录下的share/cmake-<version>/Modules目录，
另一个使我们指定的CMAKE_MODULE_PATH的所在目录。

如果Module模式搜索失败，没有找到对应的Find<LibraryName>.cmake文件，
则转入Config模式进行搜索。
它主要通过<LibraryName>Config.cmake or <lower-case-package-name>-config.cmake这两个文件来引入我们需要的库。

在cmake文件夹下新建一个FindAdd.cmake的文件。我们的目标是找到库的头文件所在目录和共享库文件的所在位置。
```
# FindAdd.cmake
# 在指定目录下寻找头文件和动态库文件的位置，可以指定多个目标路径
find_path(ADD_INCLUDE_DIR libadd.h /usr/include/ /usr/local/include ${CMAKE_SOURCE_DIR}/ModuleMode)
find_library(ADD_LIBRARY NAMES add PATHS /usr/lib/add /usr/local/lib/add ${CMAKE_SOURCE_DIR}/ModuleMode)

if (ADD_INCLUDE_DIR AND ADD_LIBRARY)
    set(ADD_FOUND TRUE)
endif (ADD_INCLUDE_DIR AND ADD_LIBRARY)
```

在CMakeLists.txt中添加
```
# 将项目目录下的cmake文件夹加入到CMAKE_MODULE_PATH中，让find_pakcage能够找到我们自定义的函数库
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake;${CMAKE_MODULE_PATH}")
add_executable(addtest addtest.cc)
find_package(ADD)
if(ADD_FOUND)
    target_include_directories(addtest PRIVATE ${ADD_INCLUDE_DIR})
    target_link_libraries(addtest ${ADD_LIBRARY})
else(ADD_FOUND)
    message(FATAL_ERROR "ADD library not found")
endif(ADD_FOUND)
```

```
find_package(Qt5 REQUIRED Core Gui Network PrintSupport Widgets Xml)
find_package(Qt5LinguistTools)
find_package(asio CONFIG) #CONFIG REQUIRED
find_package(CURL CONFIG REQUIRED)
find_package(GTest CONFIG REQUIRED)
find_package(libyuv CONFIG REQUIRED)
find_package(OpenSSL REQUIRED)
find_package(Opus CONFIG REQUIRED)
find_package(Protobuf CONFIG REQUIRED)
find_package(RapidJSON CONFIG REQUIRED)
find_package(libvpx ) #CONFIG REQUIRED
find_package(sqlite3 ) #CONFIG REQUIRED
find_package(zstd ) #CONFIG REQUIRED
find_package(mimalloc CONFIG) # Optional component
```

# 136 qt official download

https://download.qt.io/official_releases/qt/6.5/
https://download.qt.io/official_releases/qt/5.15/5.15.10/single/
https://download.qt.io/official_releases/qt/5.15/5.15.10/submodules/


# 137 use std::make_uniques in c++11

尽量使用std::make_unique
使用std::make_unique来创建std::unique_ptr智能指针有以下优点：

减少代码重复：从代码std::unique_ptr<Foo> upFoo(new Foo);和auto upFoo = std::make_unique<Foo>();
可以得知使用make_unique只需要写一次Foo就可以，更加符合软件工程中的要求。

提高异常安全性：当在函数调用中构造智能指针时，由于执行顺序的不确定性，有可能会造成资源泄露，比如对于代码：

```
#include <iostream>
#include <memory>
#include <exception>

bool priority() {
    throw std::exception();

    return true;
}

void func(std::unique_ptr<int> upNum, bool flag) {
    if (flag) {
        std::cout << *upNum << std::endl;
    }
}

int main() {
    func(std::unique_ptr<int>(new int), priority());

    return 0;
}
```

这里调用func函数时，会执行三个步骤

1. new int
2. std::unique_ptr<int>构造函数
3. priority函数
这里唯一可以确定的就是步骤1发生在步骤2之前，但步骤3的次序是不一定的，
如果步骤3在步骤1和步骤2中间执行那么就会造成内存泄漏。
但是如果使用make_unique就不会出现这个问题。

需要注意的是std::make_unique是C++14标准才引入的，所以使用C++11环境的话需要自己实现这个函数：

```
template<typename T, typename... Ts>
std::unique_ptr<T> make_unique(Ts&&... params)
{
    return std::unique_ptr<T>(new T(std::forward<Ts>(params)...));
}
```

# 138  install deb file in  linux

 sudo dpkg -i aspia-router-2.5.2-x86_64.deb

 dpkg: 处理归档 aspia-router-2.5.2-x86_64.deb (--install)时出错：
 软件包体系结构(amd64)与本机系统体系结构(arm64)不符
 在处理时有错误发生：
 aspia-router-2.5.2-x86_64.deb

```
dpkg --help
用法：dpkg [<选项> ...] <命令>

命令：
  -i|--install       <.deb 文件名> ... | -R|--recursive <目录> ...
  --unpack           <.deb 文件名> ... | -R|--recursive <目录> ...
  -A|--record-avail  <.deb 文件名> ... | -R|--recursive <目录> ...
  --configure        <软件包名>    ... | -a|--pending
  --triggers-only    <软件包名>    ... | -a|--pending
  -r|--remove        <软件包名>    ... | -a|--pending
  -P|--purge         <软件包名>    ... | -a|--pending
  -V|--verify <软件包名> ...       检查包的完整性。
  --get-selections [<表达式> ...]  把已选中的软件包列表打印到标准输出。
  --set-selections                 从标准输入里读出要选择的软件。
  --clear-selections               取消选中所有不必要的软件包。
  --update-avail <软件包文件>      替换现有可安装的软件包信息。
  --merge-avail  <软件包文件>      把文件中的信息合并到系统中。
  --clear-avail                    清除现有的软件包信息。
  --forget-old-unavail             忘却已被卸载的不可安装的软件包。
  -s|--status      <软件包名> ...  显示指定软件包的详细状态。
  -p|--print-avail <软件包名> ...  显示可供安装的软件版本。
  -L|--listfiles   <软件包名> ...  列出属于指定软件包的文件。
  -l|--list  [<表达式> ...]        简明地列出软件包的状态。
  -S|--search <表达式> ...         搜索含有指定文件的软件包。
  -C|--audit [<表达式> ...]        检查是否有软件包残损。
  --yet-to-unpack                  列出标记为待解压的软件包。
  --predep-package                 列出待解压的预依赖。
  --add-architecture    <体系结构> 添加 <体系结构> 到体系结构列表。
  --remove-architecture <体系结构> 从架构列表中移除 <体系结构>。
  --print-architecture             显示 dpkg 体系结构。
  --print-foreign-architectures    显示已启用的异质体系结构。
  --assert-<特性>                  对指定特性启用断言支持。
  --validate-<属性> <字符串>       验证一个 <属性>的 <字符串>。
  --compare-vesions <a> <关系> <b> 比较版本号 - 见下。
  --force-help                     显示本强制选项的帮助信息。
  -Dh|--debug=help                 显示有关出错调试的帮助信息。

  -?, --help                       显示本帮助信息。
      --version                    显示版本信息。

Assert 特性： support-predepends, working-epoch, long-filenames,
  multi-conrep, multi-arch, versioned-provides.

可验证的属性：pkgname, archname, trigname, version.

调用 dpkg 并带参数 -b, --build, -c, --contents, -e, --control, -I, --info,
  -f, --field, -x, --extract, -X, --vextract, --ctrl-tarfile, --fsys-tarfile
是针对归档文件的。 (输入 dpkg-deb --help 获取帮助)

选项：
  --admindir=<目录>          使用 <目录> 而非 /var/lib/dpkg。
  --root=<目录>              安装到另一个根目录下。
  --instdir=<目录>           改变安装目录的同时保持管理目录不变。
  --path-exclude=<表达式>    不要安装符合Shell表达式的路径。
  --path-include=<表达式>    在排除模式后再包含一个模式。
  -O|--selected-only         忽略没有被选中安装或升级的软件包。
  -E|--skip-same-version     忽略版本与已安装软件版本相同的软件包。
  -G|--refuse-downgrade      忽略版本早于已安装软件版本的的软件包。
  -B|--auto-deconfigure      就算会影响其他软件包，也要安装。
  --[no-]triggers            跳过或强制随之发生的触发器处理。
  --verify-format=<格式>     检查输出格式('rpm'被支持)。
  --no-debsig                不去尝试验证软件包的签名。
  --no-act|--dry-run|--simulate
                             仅报告要执行的操作 - 但是不执行。
  -D|--debug=<八进制数>      开启调试(参见 -Dhelp 或者 --debug=help)。
  --status-fd <n>            发送状态更新到文件描述符<n>。
  --status-logger=<命令>     发送状态更新到 <命令> 的标准输入。
  --log=<文件名>             将状态更新和操作信息到 <文件名>。
  --ignore-depends=<软件包>,...
                             忽略关于 <软件包> 的所有依赖关系。
  --force-...                忽视遇到的问题(参见 --force-help)。
  --no-force-...|--refuse-...
                             当遇到问题时中止运行。
  --abort-after <n>          累计遇到 <n> 个错误后中止。

可供--compare-version 使用的比较运算符有：
 lt le eq ne ge gt        (如果版本号为空，那么就认为它先于任意版本号)；
 lt-nl le-nl ge-nl gt-nl  (如果版本号为空，那么就认为它后于任意版本号)；
 < << <= = >= >> >        (仅仅是为了与主控文件的语法兼容)。
```


# 139  template in intelliJ IDEA 

## live method template
1. Settings -> Editor -> Live Templates
2. add Template Group;
3.in group add Live Template; name it '@m'
4. in Template text window,input following context:

```
/********************
 *@description   this is a brief generated by $USER$ in $PROJECT_NAME$ ,
 *@methodname    $methodname$          
 *@methodparam   $params$
 *@paramsinfo    $paramsinfo$
 *@return        $return$
 *@creator       cyclezone
 *@createtime    $date$
 *@otherNote
 ********************/
```
5. define the applicable contexts,such as  'JAVA';
6. Apply the configure;
7. in java method,press @m to autoly generate conments;

## Code template

1. Settings -> Editor -> File and Code Templates
2. Includes Tab ,add File Header;
3. input the following context into the right blank;

```
/**
 * @brief:    this is a brief information generated by ${USER} code template in ${PROJECT_NAME} ;
 *            it contains code fragment that can help reader to read the logic easily;
 *            copyright (C) 2020 - ${YEAR} ,cyclezone.cn
 * @package:  ${PACKAGE_NAME}
 * @project:  ${PROJECT_NAME}
 * @file:     ${NAME}.java
 * @author:   cyclezone
 * @create:   ${DATE} ${TIME}
 */
```


# 140  GLFW demo

```
#include <glad/gl.h>
#define GLFW_INCLUDE_NONE
#include <glfw/glfw3.h>

#include "./glfw/deps/linmath.h"

#include <stdlib.h>
#include <stdio.h>

static const struct
{
	float x, y;
	float r, g, b;
} vertices[3] =
{
	{ -0.6f, -0.4f, 1.f, 0.f, 0.f },
	{ 0.6f, -0.4f, 0.f, 1.f, 0.f },
	{ 0.f,  0.6f, 0.f, 0.f, 1.f }
};

static const char* vertex_shader_text =
"#version 110\n"
"uniform mat4 MVP;\n"
"attribute vec3 vCol;\n"
"attribute vec2 vPos;\n"
"varying vec3 color;\n"
"void main()\n"
"{\n"
"    gl_Position = MVP * vec4(vPos, 0.0, 1.0);\n"
"    color = vCol;\n"
"}\n";

static const char* fragment_shader_text =
"#version 110\n"
"varying vec3 color;\n"
"void main()\n"
"{\n"
"    gl_FragColor = vec4(color, 1.0);\n"
"}\n";

static void error_callback(int error, const char* description)
{
	fprintf(stderr, "Error: %s\n", description);
}

static void key_callback(GLFWwindow* window, int key, int scancode, int action, int mods)
{
	if (key == GLFW_KEY_ESCAPE && action == GLFW_PRESS)
		glfwSetWindowShouldClose(window, GLFW_TRUE);
}

int main(void)
{
	GLFWwindow* window;
	GLuint vertex_buffer, vertex_shader, fragment_shader, program;
	GLint mvp_location, vpos_location, vcol_location;

	glfwSetErrorCallback(error_callback);

	if (!glfwInit())
		exit(EXIT_FAILURE);

	glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 2);
	glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 0);

	window = glfwCreateWindow(640, 480, "Simple example", NULL, NULL);
	if (!window)
	{
		glfwTerminate();
		exit(EXIT_FAILURE);
	}

	glfwSetKeyCallback(window, key_callback);

	glfwMakeContextCurrent(window);
	gladLoadGL(glfwGetProcAddress);
	glfwSwapInterval(1);

	// NOTE: OpenGL error checks have been omitted for brevity

	glGenBuffers(1, &vertex_buffer);
	glBindBuffer(GL_ARRAY_BUFFER, vertex_buffer);
	glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);

	vertex_shader = glCreateShader(GL_VERTEX_SHADER);
	glShaderSource(vertex_shader, 1, &vertex_shader_text, NULL);
	glCompileShader(vertex_shader);

	fragment_shader = glCreateShader(GL_FRAGMENT_SHADER);
	glShaderSource(fragment_shader, 1, &fragment_shader_text, NULL);
	glCompileShader(fragment_shader);

	program = glCreateProgram();
	glAttachShader(program, vertex_shader);
	glAttachShader(program, fragment_shader);
	glLinkProgram(program);

	mvp_location = glGetUniformLocation(program, "MVP");
	vpos_location = glGetAttribLocation(program, "vPos");
	vcol_location = glGetAttribLocation(program, "vCol");

	glEnableVertexAttribArray(vpos_location);
	glVertexAttribPointer(vpos_location, 2, GL_FLOAT, GL_FALSE,
		sizeof(vertices[0]), (void*)0);
	glEnableVertexAttribArray(vcol_location);
	glVertexAttribPointer(vcol_location, 3, GL_FLOAT, GL_FALSE,
		sizeof(vertices[0]), (void*)(sizeof(float) * 2));

	while (!glfwWindowShouldClose(window))
	{
		float ratio;
		int width, height;
		mat4x4 m, p, mvp;

		glfwGetFramebufferSize(window, &width, &height);
		ratio = width / (float)height;

		glViewport(0, 0, width, height);
		glClear(GL_COLOR_BUFFER_BIT);

		mat4x4_identity(m);
		mat4x4_rotate_Z(m, m, (float)glfwGetTime());
		mat4x4_ortho(p, -ratio, ratio, -1.f, 1.f, 1.f, -1.f);
		mat4x4_mul(mvp, p, m);

		glUseProgram(program);
		glUniformMatrix4fv(mvp_location, 1, GL_FALSE, (const GLfloat*)mvp);
		glDrawArrays(GL_TRIANGLES, 0, 3);

		glfwSwapBuffers(window);
		glfwPollEvents();
	}

	glfwDestroyWindow(window);

	glfwTerminate();
	exit(EXIT_SUCCESS);
}

```

# 141  error LNK2001 for c file 

when compiling files like ' triangle.c ' ,
crush the following error:

 error LNK2001: 无法解析的外部符号 "void __cdecl triangulate(char *,struct triangulateio *,struct triangulateio *,struct triangulateio *)" (?triangulate@@YAXPEADPEAUtriangulateio@@11@Z)
1>multipolygon.obj : error LNK2001: 无法解析的外部符号 "void __cdecl trifree(int *)" (?trifree@@YAXPEAH@Z)

backkgroud:
we has the function declaration and defination;
we has compile the c file;

SOLUTION:
1. open c file property sheet;
2. in left tree ,select c/c++  --> advance ,
3. change the item 'compile as ' 编译为 C++ 代码 (/TP)

DONOT use '默认值' or '编译为 C 代码 (/TC)'

# 142 NamedPipeServerStream and NamedPipeClientStream

```

public class XVersion
{
	/// <summary>
	/// 获取得到当前X文件的版本号，当X文件对应数据结构发生变化，请更新该值。
	/// </summary>
	/// <returns></returns>
	public static string GetVersion()
	{
		return "202307";
	}

	public static string GetNamedPipeClientStreamName()
	{
		return "server_NamedPipe" + GetVersion();
	}
}


try
{
	NamedPipeServerStream pipeServer =
	new NamedPipeServerStream(XVersion.GetNamedPipeClientStreamName(), PipeDirection.InOut, 1);

	pipeServer.WaitForConnection();
	pipeServer.ReadMode = PipeTransmissionMode.Byte;
	using (StreamReader render = new StreamReader(pipeServer))
	{
		string strStream = render.ReadToEnd();
		String[] Lines = strStream.Split('|');
		if (Lines.Length == 4)
		{
			//退出当前线程
			if (Lines[0] == "exit")
				break;
			Semaphore ConvertOut = null;
			try
			{
				ConvertOut = Semaphore.OpenExisting(Lines[2].Replace('\\', '/'));
			}
			catch (WaitHandleCannotBeOpenedException)
			{
				Console.WriteLine("Semaphore does not exist.");
			}
			catch (UnauthorizedAccessException ex)
			{
				Console.WriteLine("Unauthorized access: {0}", ex.Message);
			}

			try
			{
				if (Lines[0] == "xml")
				{
					convertOut.Convert(Lines[1], Lines[2], Lines[3]);
				}
				else if (Lines[0] == "style")
				{
				}
			}
			catch(Exception e)
			{
				Console.WriteLine("Unauthorized access: {0}", e.Message);
			}
			if (ConvertOut != null)
			{
				ConvertOut.Release(1);
			}
		}
	}
}
catch
{
}
}

```

```

try
{
	Semaphore ConvertOutOkMessage = new Semaphore(0, 1, strDstPath.Replace('\\', '/'));

	using (NamedPipeClientStream pipeClient =
	new NamedPipeClientStream("localhost", XVersion.GetNamedPipeClientStreamName(), PipeDirection.InOut, PipeOptions.None, TokenImpersonationLevel.None))
	{
		pipeClient.Connect();
		using (StreamWriter swriter = new StreamWriter(pipeClient))
		{
			string strStream = strType + "|" + strSrcPath + "|" + strDstPath + ;
			swriter.Write(strStream);
			swriter.Flush();
		}
	}

	while (true)
	{
		if (ConvertOutOkMessage.WaitOne(1000))
			return true;
		if (!IsConvertOutOnWorking())
		{
			log.AppendLog(ConvertLog.LogLevel.Error, string.Format("插件未运行，或者使用了不匹配的插件，请启动({0}版本)插件!", XVersion.GetVersion()));
			return false;
		}
		else
		{
			log.AppendLog(ConvertLog.LogLevel.Log, "解码中...");
		}
	}
}
catch
{
}

```


# 143  Geotiff_Information

```
Geotiff_Information:
   Version: 1
   Key_Revision: 1.0
   Tagged_Information:
      ModelTiepointTag (2,3):
         0                 0                 0
         364150.131765544  3483476.70992806  0
      ModelPixelScaleTag (1,3):
         0.433112592500314 0.433112592500408 0
      End_Of_Tags.
   Keyed_Information:
      GTModelTypeGeoKey (Short,1): ModelTypeProjected
      GTRasterTypeGeoKey (Short,1): RasterPixelIsArea
      GTCitationGeoKey (Ascii,22): "WGS 84 / UTM zone 48N"
      GeogCitationGeoKey (Ascii,7): "WGS 84"
      GeogAngularUnitsGeoKey (Short,1): Angular_Degree
      ProjectedCSTypeGeoKey (Short,1): PCS_WGS84_UTM_zone_48N
      ProjLinearUnitsGeoKey (Short,1): Linear_Meter
      End_Of_Keys.
   End_Of_Geotiff.

PCS = 32648 (WGS 84 / UTM zone 48N)
Projection = 16048 (UTM zone 48N)
Projection Method: CT_TransverseMercator
   ProjNatOriginLatGeoKey: 0.000000 (  0d 0' 0.00"N)
   ProjNatOriginLongGeoKey: 105.000000 (105d 0' 0.00"E)
   ProjScaleAtNatOriginGeoKey: 0.999600
   ProjFalseEastingGeoKey: 500000.000000 m
   ProjFalseNorthingGeoKey: 0.000000 m
GCS: 4326/WGS 84
Datum: 6326/World Geodetic System 1984
Ellipsoid: 7030/WGS 84 (6378137.00,6356752.31)
Prime Meridian: 8901/Greenwich (0.000000/  0d 0' 0.00"E)
Projection Linear Units: 9001/metre (1.000000m)

PROJ.4 Definition: +proj=utm +zone=48 +ellps=WGS84 +units=m

Corner Coordinates:
Upper Left    (  364150.132, 3483476.710)proj_create_operation_factory_context:
Cannot find proj.db
pj_obj_create: Cannot find proj.db
  (103d34'11.51"E, 31d28'41.22"N)
Lower Left    (  364150.132, 3482654.662)proj_create_operation_factory_context:
Cannot find proj.db
pj_obj_create: Cannot find proj.db
  (103d34'11.92"E, 31d28'14.53"N)
Upper Right   (  365126.368, 3483476.710)proj_create_operation_factory_context:
Cannot find proj.db
pj_obj_create: Cannot find proj.db
  (103d34'48.50"E, 31d28'41.63"N)
Lower Right   (  365126.368, 3482654.662)proj_create_operation_factory_context:
Cannot find proj.db
pj_obj_create: Cannot find proj.db
  (103d34'48.91"E, 31d28'14.94"N)
Center        (  364638.250, 3483065.686)proj_create_operation_factory_context:
Cannot find proj.db
pj_obj_create: Cannot find proj.db
  (103d34'30.21"E, 31d28'28.08"N)
  
```

# 144  open html with the default explore

```
private void OpenHtml(string str)
{
	try
	{
		//通过注册表获取默认浏览器
		Microsoft.Win32.RegistryKey key = Microsoft.Win32.Registry.ClassesRoot.OpenSubKey(@"http\shell\open\command\");
		string s = key.GetValue("").ToString();
		string browserpath = null;
		if (s.StartsWith("\""))
		{
			browserpath = s.Substring(1, s.IndexOf('\"', 1) - 1);
		}
		else
		{
			browserpath = s.Substring(0, s.IndexOf(" "));
		}
		while (str.IndexOf('\\') > 0)
		{
			str = str.Replace('\\', '/');
		}
		string strTar = "\"" + str + "\""; /// for space in path
		System.Diagnostics.Process.Start(browserpath, strTar);
	}
	catch (System.Exception ex)
	{
		MessageBox.Show(ex.Message.ToString());
	}


}
```

# 145  make table in html 

```

private string MakeColumn(string key, string val, bool bShow = true)
{

	string strInfo = "<tr>" 
	+ "<th width=\"100\" height=\"20\" scope=\"col\">" + key + "</th>"
	+ "<th height=\"20\" colspan=\"3\" align=\"left\" scope=\"col\">" + val + "</th>"
	+ "</tr>";
	return strInfo;
}

private string MakeTablehead(string producer = "cycle", string version = "10060520")
{

	string strInfo = "<tr>"
			+ "<th height=\"50\" colspan=\"3\" scope=\"col\" ><strong style=\"font-size: 24px\">数据迁移报告 </strong></th>"
			+ "</tr>"
			+"<tr>"
			+"<th width=\"100\" height=\"20\" scope=\"col\">" + producer + "</th>"
			+"<th width=\"266\" height=\"20\" scope=\"col\">版本：" + version +"</th>"
			+"<th width=\"287\" height=\"20\" scope=\"col\">日期：" + System.DateTime.Now.ToString("yyyy.MM.dd.HH:mm:ss") + "</th>"
			+"</tr>";
	return strInfo;
}

private string MakeTableTail(bool bConvertOK = true)
{
	string strCvt = bConvertOK ? "<br>已完成迁移。" : "<br>出现迁移问题,请检查。";
	string strOverview = GetOverview();
	string strInfo = "<th height=\"30\" scope=\"col\">总结</th>"
				+ "<th colspan=\"2\" align=\"left\" scope=\"col\">" + strOverview + strCvt + "</th>"
				+ "</tr>"
				+ "<tr>"
				+ "<th height=\"30\" colspan=\"2\" align=\"left\" scope=\"col\">&nbsp;</th>"
				+ "<th height=\"30\" align=\"center\" scope=\"col\">创建者: cycle 制图转换插件</th>"
				+ "</tr>";
	return strInfo;
}

private string MakeTable()
{
	string strinfo = "";
	string strOverview = GetOverview();
	strinfo += MakeColumn("概述", strOverview);
	for (int i = 0; i < mMxd.Maps.Count; i++)
	{
		Arc_Map map = mMxd.Maps[i];

		strinfo += MakeColumn("地图", string.Format("地图 {0} 有 {1:d} 个图层，如下：", map.Name, map.Layers.Count));
		
		for (int m = 0; m < map.Layers.Count; m++)
		{
			ArcLayer layr = map.Layers[m];
			strinfo += checkLayer(layr); 
		}
		
	}
	return strinfo;
}

 string table = "<table width=\"1024\" border=\"1\" style=\"margin:auto\" bordercolor=\"#000000\" class=\"gridtable\">"
                    + "<tbody>"
                    + MakeTablehead()
                    + MakeColumn("数据源", DocumentFilename)
                    + MakeColumn("迁移结果", str)
                    + MakeTable()
                    + MakeTableTail()
                    +"</tbody>"
                    +"</table>";
					
   string style = "<style type=\"text/css\">"
		+ "table.gridtable { font-family: verdana,arial,sans-serif;font-size:15px;color:#333333;border-width: 1px;border-color: #666666;border-collapse: collapse;}"
		+ "table.gridtable th {border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #dedede;}"
		+ "table.gridtable td {border-width: 1px;padding: 8px;border-style: solid;border-color: #666666;background-color: #ffffff;}"
		+ "</style>";

	string body = "<body>" +
				table +
				"</body>";
	string head = "<head>" +
			"<meta charset='UTF-8'>" +
			"<meta name='viewport' content='width=device-width, initial-scale=1.0'>" +
			"<title>数据转换日志</title>" +
			script +
			"</head>";
	string html = "<!DOCTYPE html>" +
			"<html lang='en'>" +
			head +
			style+
			body +
			"</html>";
```


# 146  create html 

```
  void Create(string strFile)
  {
	string strTempFile = strFile.Substring(0, strFile.LastIndexOf('.')) + ".html";
	string path = strFile.Substring(0, strFile.LastIndexOf('\\'));
	if (Directory.Exists(path) == false)//如果不存在就创建file文件夹
	{
		Directory.CreateDirectory(path);
	}

	FileStream fs1 = new FileStream(strTempFile, System.IO.FileMode.Create, FileAccess.Write);//创建写入文件 
	StreamWriter sw = new StreamWriter(fs1);

	sw.WriteLine(html);//开始写入值
	sw.Close();
	fs1.Close();

  }

```

# 147   nm -D  /opt/apps/files/program/libQt5Gui.so | grep QTextEngine

```
nm -D /home/os/program/libQt5Gui.so

ldd -r /opt/apps/files/program/libQt5Gui.so
        linux-vdso.so.1 (0x000000200117e000)
        libQt5Core.so.5 => /lib/aarch64-linux-gnu/libQt5Core.so.5 (0x00000020016fa000)
        libpthread.so.0 => /lib/aarch64-linux-gnu/libpthread.so.0 (0x0000002001c2a000)
        libz.so.1 => /lib/aarch64-linux-gnu/libz.so.1 (0x0000002001c5a000)
        libstdc++.so.6 => /lib/aarch64-linux-gnu/libstdc++.so.6 (0x0000002001c83000)
        libm.so.6 => /lib/aarch64-linux-gnu/libm.so.6 (0x0000002001e68000)
        libgcc_s.so.1 => /lib/aarch64-linux-gnu/libgcc_s.so.1 (0x0000002001f13000)
        libc.so.6 => /lib/aarch64-linux-gnu/libc.so.6 (0x0000002001f37000)
        /lib/ld-linux-aarch64.so.1 (0x000000200115c000)
        libicui18n.so.66 => /lib/aarch64-linux-gnu/libicui18n.so.66 (0x00000020020aa000)
        libicuuc.so.66 => /lib/aarch64-linux-gnu/libicuuc.so.66 (0x0000002002396000)
        libdl.so.2 => /lib/aarch64-linux-gnu/libdl.so.2 (0x0000002002582000)
        libpcre2-16.so.0 => /lib/aarch64-linux-gnu/libpcre2-16.so.0 (0x0000002002596000)
        libdouble-conversion.so.3 => /lib/aarch64-linux-gnu/libdouble-conversion.so.3 (0x0000002002619000)
        libglib-2.0.so.0 => /lib/aarch64-linux-gnu/libglib-2.0.so.0 (0x000000200263c000)
        libicudata.so.66 => /lib/aarch64-linux-gnu/libicudata.so.66 (0x0000002002776000)
        libpcre.so.3 => /lib/aarch64-linux-gnu/libpcre.so.3 (0x0000002004245000)

 nm -D  /opt/apps/files/program/libQt5Gui.so | grep QTextEngine
0000000000240940 T _ZN11QTextEngine10freeMemoryEv
0000000000240b30 T _ZN11QTextEngine10invalidateEv
000000000023b838 T _ZN11QTextEngine10LayoutData10reallocateEi
000000000023af88 T _ZN11QTextEngine10LayoutDataC1ERK7QStringPPvi
000000000023ace0 T _ZN11QTextEngine10LayoutDataC1Ev
000000000023af88 T _ZN11QTextEngine10LayoutDataC2ERK7QStringPPvi
000000000023ace0 T _ZN11QTextEngine10LayoutDataC2Ev
000000000023b188 T _ZN11QTextEngine10LayoutDataD1Ev
000000000023b188 T _ZN11QTextEngine10LayoutDataD2Ev
0000000000248ac8 T _ZN11QTextEngine10setFormatsERK7QVectorIN11QTextLayout11FormatRangeEE
0000000000240460 T _ZN11QTextEngine11addOverlineEP8QPainterRK6QLineF
000000000023a088 T _ZN11QTextEngine11bidiReorderEiPKhPi
0000000000240420 T _ZN11QTextEngine12addStrikeOutEP8QPainterRK6QLineF
00000000002403e0 T _ZN11QTextEngine12addUnderlineEP8QPainterRK6QLineF
00000000002488f0 T _ZN11QTextEngine12indexFormatsEv
00000000002408e8 T _ZN11QTextEngine13clearLineDataEv
0000000000240b80 T _ZN11QTextEngine14setPreeditAreaEiRK7QString
0000000000247b50 T _ZN11QTextEngine15beginningOfLineEi
00000000002407b8 T _ZN11QTextEngine15drawDecorationsEP8QPainter
000000000023a920 T _ZN11QTextEngine15FontEngineCacheC1Ev
000000000023a920 T _ZN11QTextEngine15FontEngineCacheC2Ev
000000000023da58 T _ZN11QTextEngine16adjustUnderlinesEPNS_14ItemDecorationES1_dd
0000000000240538 T _ZN11QTextEngine16adjustUnderlinesEv
00000000002404a0 T _ZN11QTextEngine16clearDecorationsEv
000000000023d8f0 T _ZN11QTextEngine16getClusterLengthEPtPK15QCharAttributesiiiPi
000000000023d7e8 T _ZN11QTextEngine16offsetInLigatureEPK11QScriptItemiii
00000000002402a0 T _ZN11QTextEngine17addItemDecorationEP8QPainterRK6QLineFP7QVectorINS_14ItemDecorationEE
0000000000245fb8 T _ZN11QTextEngine17leadingSpaceWidthERK11QScriptLine


```


# 148    __attribute__ ((visibility("default"))


```
GNU C 的一大特色就是attribute 机制。
试想这样的情景，程序调用某函数A，A函数存在于两个动态链接库liba.so,libb.so中，并且程序执行需要链接这两个库，此时程序调用的A函数到底是来自于a还是b呢？
这取决于链接时的顺序，比如先链接liba.so，这时候通过liba.so的导出符号表就可以找到函数A的定义，并加入到符号表中，链接libb.so的时候，符号表中已经存在函数A，就不会再更新符号表，所以调用的始终是liba.so中的A函数。
为了避免这种混乱，所以使用

__attribute__((visibility("default")))  //默认，设置为：default之后就可以让外面的类看见了。
__attribute__((visibility("hideen")))  //隐藏
设置这个属性。

isibility用于设置动态链接库中函数的可见性，将变量或函数设置为hidden，则该符号仅在本so中可见，在其他库中则不可见。

g++在编译时，可用参数-fvisibility指定所有符号的可见性（不加此参数时默认外部可见，参考man g++中-fvisibility部分）；若需要对特定函数的可见性进行设置，需在代码中使用attribute设置visibility属性。

编写大型程序时，可用-fvisibility=hidden设置符号默认隐藏，针对特定变量和函数，在代码中使用attribute ((visibility("default")))另该符号外部可见，这种方法可用有效避免so之间的符号冲突

```
```
/* GCC visibility */
#if defined(__GNUC__) && __GNUC__ >= 4
#define WL_EXPORT __attribute__ ((visibility("default")))
#else
#define WL_EXPORT
#endif

#if defined(_WIN32)
    #if defined(COMPILING_SHARED_META)
        #define META_API __declspec(dllexport)
        #define META_DOC_API __declspec(dllexport)
	#elif defined(USING_SHARED_META)
		#define META_API __declspec(dllimport)
        #define META_DOC_API __declspec(dllimport)
    #else
        #define META_API
        #define META_DOC_API
    #endif
    /* META_LOCAL doesn't mean anything on win32, it's to exclude
     * symbols from the export table with gcc4. */
    #define META_LOCAL
#else
    #if defined(META_HAVE_GCC_SYMBOL_VISIBILITY)
        /* Forces inclusion of a symbol in the symbol table, so
           software outside the current library can use it. */
        #define META_API __attribute__ ((visibility("default")))
        #define META_DOC_API __attribute__ ((visibility("default")))
        /* Within a section exported with META_API, forces a symbol to be
           private to the library / app. Good for private members. */
        #define META_LOCAL __attribute__ ((visibility("hidden")))
        /* Forces even stricter hiding of methods/functions. The function must
         * absolutely never be called from outside the module even via a function
         * pointer.*/
        #define META_INTERNAL __attribute__ ((visibility("internal")))
    #else
        #define META_API
        #define META_DOC_API
        #define META_LOCAL
        #define META_INTERNAL
    #endif
#endif


```
# 149 recompile with -fPIC

1. fpic 和 fPIC 区别

在64位下编译动态库的时候，经常会遇到下面的错误
```
/usr/bin/ld: /tmp/ccQ1dkqh.o: relocation R_X86_64_32 against `a local symbol' can not be used when making a shared object; recompile with -fPIC
```

```
Use -fPIC or -fpic to generate code. Whether to use -fPIC or -fpic to generate code is target-dependent. The -fPIC choice always works, but may produce larger code than -fpic (mnenomic to remember this is that PIC is in a larger case, so it may produce larger amounts of code). Using -fpic option usually generates smaller and faster code, but will have platform-dependent limitations, such as the number of globally visible symbols or the size of the code. The linker will tell you whether it fits when you create the shared library. When in doubt, I choose -fPIC, because it always works.

-fPIC： 总是能够工作，但可能产生的代码较大
-fpic： 通常能产生更快更小的代码，但有平台限制。
```

# 150 pthread_create and pthread_exit

## 1. 线程创建

　　在进程被创建时，系统会为其创建一个主线程，而要在进程中创建新的线程，则可以调用pthread_create函数：
```
	#include <pthread.h>
	int pthread_create(pthread_t *thread, pthread_attr_t *attr, void *(*start_routine)(void *), void *arg);
　　参数说明：

	thread：指向pthread_create类型的指针，用于引用新创建的线程。
	attr：用于设置线程的属性，一般不需要特殊的属性，所以可以简单地设置为NULL。
	start_routine：传递新线程所要执行的函数地址。
	arg：新线程所要执行的函数的参数。
　　返回值：
　　调用如果成功，则返回值是0；如果失败则返回错误代码。
```
　　每个线程都有自己的线程ID，以便在进程内区分。线程ID在pthread_create调用时回返给创建线程的调用者；一个线程也可以在创建后使用pthread_self()调用获取自己的线程ID： 

pthread_self (void);

## 2. 线程退出

　　线程的退出方式有三种：

（1）执行完成后隐式退出；

（2）由线程本身显示调用pthread_exit 函数退出；
   ```
   pthread_exit (void * retval);
   ```
（3）被其他线程用pthread_cance函数终止：
  ```
  pthread_cancel (pthread_t thread);
  ```

　　如果一个线程要等待另一个线程的终止，可以使用pthread_join函数，该函数的作用是调用pthread_join的线程将被挂起直到线程ID为参数thread的线程终止：
  ```
  pthread_join (pthread_t thread, void** threadreturn);
  ```
## 3.  Compare 

    
	事项	       WIN32	                Linux	                     VxWorks
	线程创建	    CreateThread	        pthread_create	         taskSpawn
	线程终止(byself)  ExitThread            pthread_exit             exit
	线程终止(byOther)  TerminateThread      pthread_cance            taskDelete
	获取线程ID	     GetCurrentThreadId  	pthread_self	         taskIdSelf
	创建互斥	     CreateMutex	        pthread_mutex_init	     semMCreate
	获取互斥	     WaitForMultipleObjects	pthread_mutex_lock	     semTake
	释放互斥	     ReleaseMutex	        phtread_mutex_unlock	 semGive
	创建信号量	     CreateSemaphore	    sem_init	             semBCreate
	等待信号量	     WaitForSingleObject	sem_wait	             semTake
	释放信号量	     ReleaseSemaphore	    sem_post	             semGive
  

## 4. Demo

  ```
	#include <stdlib.h>
	#include <stdio.h>
	#include <errno.h>
	#include <pthread.h>
	static void pthread_func_1 (void);
	static void pthread_func_2 (void);

	int main (int argc, char** argv)
	{
		pthread_t pt_1 = 0;
		pthread_t pt_2 = 0;
		pthread_attr_t atrr = {0};
		int ret = 0;

		/*初始化属性线程属性*/
		pthread_attr_init (&attr);
		pthread_attr_setscope (&attr, PTHREAD_SCOPE_SYSTEM);
		pthread_attr_setdetachstate (&attr, PTHREAD_CREATE_DETACHED);

		ret = pthread_create (&pt_1, &attr, pthread_func_1, NULL);
		if (ret != 0)
		{
			perror ("pthread_1_create");
		}

		ret = pthread_create (&pt_2, NULL, pthread_func_2, NULL);
		if (ret != 0)
		{
			perror ("pthread_2_create");
		}

		pthread_join (pt_2, NULL);

		return 0;
	}

	static void pthread_func_1 (void)
	{
		int i = 0;

		for (; i < 6; i++)
		{
			printf ("This is pthread_1.\n");

			if (i == 2)
			{
			pthread_exit (0);
			}
		}

		return;
	}

	static void pthread_func_2 (void)
	{
		int i = 0;

		for (; i < 3; i ++)
		{
			printf ("This is pthread_2.\n");
		}

		return;
	}
  ```

# 151 PostgreHelper gis function
  
 ## 1. some SQL command
  ```
  "SELECT ST_Distance(ST_Transform(ST_SetSRID(ST_MakePoint({0}),4326), 26986),
                      ST_Transform(ST_SetSRID(ST_MakePoint({1}),4326), 26986)) AS Distance"

  SELECT {0} AS Index, st_dwithin(ST_Transform(ST_SetSRID(ST_MakePoint({0}),4326), 26986),
                                  ST_Transform(ST_SetSRID(ST_MakePoint({1}),4326), 26986),{2}) AS Status

   union all SELECT {0} AS Index, st_dwithin(ST_Transform(ST_SetSRID(ST_MakePoint({0}),4326), 26986)
				                            ,ST_Transform(ST_SetSRID(ST_MakePoint({1}),4326), 26986),{2}) AS Status

   select ST_Contains(St_Astext(ST_Buffer(geography(ST_GeomFromText('LINESTRING({0})')),{1}))
                     ,st_astext(geography(ST_GeomFromText('POINT({2})')))) AS Status
	
	select {0} AS Index, ST_Contains(St_Astext(ST_Buffer(geography(ST_GeomFromText('LINESTRING({1})')),{2}))
                                    ,st_astext(geography(ST_GeomFromText('POINT({3})')))) AS Status

	SELECT ST_Contains( ST_MakePolygon(ST_GeomFromText('LINESTRING({0}) '))
                      ,st_point({1})) AS Status
  ```
 
  ## 2. MainGISFunction for PostgreSQL
  ```
   public class MainGISFunction
    {
        PostgreHelper dbHelper = new PostgreHelper();
        /// <summary>
        /// 判断两个经纬度点之间的距离
        /// 参数格式：经度,纬度
        /// </summary>
        /// <param name="lonlat1">经纬度1</param>
        /// <param name="lonlat2">经纬度2</param>
        /// <returns>直线距离（单位m）/null</returns>
        public DataTable GetPointDistance(string lonlat1, string lonlat2)
        {
            try
            {
                string strsql = string.Format(@"SELECT ST_Distance(ST_Transform(ST_SetSRID(ST_MakePoint({0}),4326), 26986)
				,ST_Transform(ST_SetSRID(ST_MakePoint({1}),4326), 26986)) AS Distance", lonlat1, lonlat2);
                return dbHelper.ExecuteQuery(strsql).Tables[0];
 
            }
            catch (Exception ex)
            {
                Log.Instance.Error("GetPointDistance:" + ex.Message);
                return null;
            }
        }
        /// <summary>
        /// 查询点与点之间相距小于某个值的集合
        /// </summary>
        /// <param name="lonlat"></param>
        /// <param name="points"></param>
        /// <param name="distance"></param>
        /// <returns></returns>
        public DataTable GetPointsDwithin(string lonlat, List<string> points, int distance)
        {
            try
            {
                StringBuilder sbsql = new StringBuilder();
                for (int i = 0; i < points.Count; i++)
                {
                    if (i == 0)
                    {
                        sbsql.AppendFormat(@"SELECT {0} AS Index, st_dwithin(ST_Transform(ST_SetSRID(ST_MakePoint({0}),4326), 26986)
				    ,ST_Transform(ST_SetSRID(ST_MakePoint({1}),4326), 26986),{2}) AS Status", lonlat, points[i], distance);
                    }
                    else
                    {
                        sbsql.AppendFormat(@" union all SELECT {0} AS Index, st_dwithin(ST_Transform(ST_SetSRID(ST_MakePoint({0}),4326), 26986)
				    ,ST_Transform(ST_SetSRID(ST_MakePoint({1}),4326), 26986),{2}) AS Status", lonlat, points[i], distance);
                    }
                }
                DataSet ds = dbHelper.ExecuteQuery(sbsql.ToString());
                DataRow[] rows = ds.Tables[0].Select("Status='true'");
                if (rows.Length > 0)
                {
                    DataTable dtNew = ds.Tables[0].Clone();
                    for (int i = 0; i < rows.Length; i++)
                    {
                        dtNew.Rows.Add(rows[i].ItemArray);
                    }
                    return dtNew;
                }
                else
                {
                    return null;
                }
            }
            catch (Exception ex)
            {
                Log.Instance.Error("GetPointOnLines:" + ex.Message);
                return null;
            }
        }
 
        /// <summary>
        /// 判断某个经纬度点是否在某条路线上
        /// </summary>
        /// <param name="lonlat">经纬度点</param>
        /// <param name="line">路线经纬度集合</param>
        /// <param name="buffer">缓冲区域，及在这个缓冲区域的点都可以判断为在这条路上，可以称作道路边缘到中心距离的宽度</param>
        /// <returns>ture/false/null</returns>
        public DataTable GetPointOnTheLine(string lonlat, List<string> line, int buffer)
        {
            try
            {
                StringBuilder sbline = new StringBuilder();
                for (int i = 0; i < line.Count; i++)
                {
                    sbline.Append(line[i].Replace(',', ' ') + ",");
                }
                string strsql = string.Format(@"select ST_Contains(St_Astext(ST_Buffer(geography(ST_GeomFromText('LINESTRING({0})')),{1}))
                    ,st_astext(geography(ST_GeomFromText('POINT({2})')))) AS Status", sbline.ToString().Trim(','), buffer, lonlat.Replace(',', ' '));
                return dbHelper.ExecuteQuery(strsql).Tables[0];
            }
            catch (Exception ex)
            {
                Log.Instance.Error("GetPointOnTheLine:" + ex.Message);
                return null;
            }
        }
 
        /// <summary>
        /// 判断某个经纬度点是否在这些路线上的其中一条
        /// </summary>
        /// <param name="lonlat">经纬度点</param>
        /// <param name="lines">多条路线经纬度集合</param>
        /// <param name="buffer">缓冲区域，及在这个缓冲区域的点都可以判断为在这条路上，可以称作道路边缘到中心距离的宽度</param>
        /// <returns>ture/false/null</returns>
        public DataTable GetPointOnOneLine(string lonlat, List<List<string>> lines, int buffer)
        {
            try
            {
                StringBuilder sblines = new StringBuilder();
                for (int i = 0; i < lines.Count; i++)
                {
                    StringBuilder sbline = new StringBuilder();
                    for (int j = 0; j < lines[i].Count; j++)
                    {
                        sbline.Append(lines[i][j].Replace(',', ' ') + ",");
                    }
                    sblines.Append("(" + sbline.ToString().Trim(',') + "),");
                }
                string strsql = string.Format(@"select ST_Contains(St_Astext(ST_Buffer(geography(ST_GeomFromText('MULTILINESTRING({0})')),{1}))
                    ,st_astext(geography(ST_GeomFromText('POINT({2})')))) AS Status", sblines.ToString().Trim(','), buffer, lonlat.Replace(',', ' '));
                return dbHelper.ExecuteQuery(strsql).Tables[0];
            }
            catch (Exception ex)
            {
                Log.Instance.Error("GetPointOnOneLine:" + ex.Message);
                return null;
            }
        }
 
 
        /// <summary>
        /// 判断某个经纬度点是否在这些路线上的其中一条
        /// </summary>
        /// <param name="lonlat">经纬度点</param>
        /// <param name="lines">多条路线经纬度集合</param>
        /// <param name="buffer">缓冲区域，及在这个缓冲区域的点都可以判断为在这条路上，可以称作道路边缘到中心距离的宽度</param>
        /// <returns>ture/false/null</returns>
        public DataTable myGetPointOnOneLine(string lonlat, List<string> lines, int buffer)
        {
            try
            {
                StringBuilder sbline = new StringBuilder();
                for (int i = 0; i < lines.Count; i++)
                {
                    sbline.Append(lines[i].Replace(',', ' ') + ",");
                }
                string strsql = string.Format(@"select ST_Contains(St_Astext(ST_Buffer(geography(ST_GeomFromText('LINESTRING({0})')),{1}))
                    ,st_astext(geography(ST_GeomFromText('POINT({2})')))) AS Status", sbline.ToString().Trim(','), buffer, lonlat.Replace(',', ' '));
                return dbHelper.ExecuteQuery(strsql).Tables[0];
            }
            catch (Exception ex)
            {
                Log.Instance.Error("myGetPointOnOneLine:" + ex.Message);
                return null;
            }
        }
 
        /// <summary>
        /// 判断某个经纬度点所在的路线
        /// </summary>
        /// <param name="lonlat"></param>
        /// <param name="lines"></param>
        /// <param name="buffer"></param>
        /// <returns>路线索引集合/null</returns>
        public DataTable GetPointOnLines(string lonlat, List<List<string>> lines, int buffer)
        {
            try
            {
                StringBuilder sbsql = new StringBuilder();
                for (int i = 0; i < lines.Count; i++)
                {
                    StringBuilder sbline = new StringBuilder();
                    for (int j = 0; j < lines[i].Count; j++)
                    {
                        sbline.Append(lines[i][j].Replace(',', ' ') + ",");
                    }
 
                    if (i == 0)
                    {
                        sbsql.AppendFormat(@"select {0} AS Index, ST_Contains(St_Astext(ST_Buffer(geography(ST_GeomFromText('LINESTRING({1})')),{2}))
                    ,st_astext(geography(ST_GeomFromText('POINT({3})')))) AS Status", i, sbline.ToString().Trim(','), buffer, lonlat.Replace(',', ' '));
                    }
                    else
                    {
                        sbsql.AppendFormat(@" union all select {0} AS Index, ST_Contains(St_Astext(ST_Buffer(geography(ST_GeomFromText('LINESTRING({1})')),{2}))
                    ,st_astext(geography(ST_GeomFromText('POINT({3})')))) AS Status", i, sbline.ToString().Trim(','), buffer, lonlat.Replace(',', ' '));
                    }
                }
                DataSet ds = dbHelper.ExecuteQuery(sbsql.ToString());
                DataRow[] rows = ds.Tables[0].Select("Status='true'");
                if (rows.Length > 0)
                {
                    DataTable dtNew = ds.Tables[0].Clone();
                    for (int i = 0; i < rows.Length; i++)
                    {
                        dtNew.Rows.Add(rows[i].ItemArray);
                    }
                    return dtNew;
                }
                else
                {
                    return null;
                }
            }
            catch (Exception ex)
            {
                Log.Instance.Error("GetPointOnLines:" + ex.Message);
                return null;
            }
        }
 
        /// <summary>
        /// 判断某个经纬度点所在的路线
        /// </summary>
        /// <param name="lonlat"></param>
        /// <param name="lines"></param>
        /// <param name="buffer"></param>
        /// <returns>路线索引集合/null</returns>
        public DataTable GetPointOnLinesByWidth(string lonlat, List<List<string>> lines, List<int> buffer)
        {
            try
            {
                StringBuilder sbsql = new StringBuilder();
                for (int i = 0; i < lines.Count; i++)
                {
                    StringBuilder sbline = new StringBuilder();
                    for (int j = 0; j < lines[i].Count; j++)
                    {
                        sbline.Append(lines[i][j].Replace(',', ' ') + ",");
                    }
 
                    if (i == 0)
                    {
                        sbsql.AppendFormat(@"select {0} AS Index, ST_Contains(St_Astext(ST_Buffer(geography(ST_GeomFromText('LINESTRING({1})')),{2}))
                    ,st_astext(geography(ST_GeomFromText('POINT({3})')))) AS Status", i, sbline.ToString().Trim(','), buffer[i], lonlat.Replace(',', ' '));
                    }
                    else
                    {
                        sbsql.AppendFormat(@" union all select {0} AS Index, ST_Contains(St_Astext(ST_Buffer(geography(ST_GeomFromText('LINESTRING({1})')),{2}))
                    ,st_astext(geography(ST_GeomFromText('POINT({3})')))) AS Status", i, sbline.ToString().Trim(','), buffer[i], lonlat.Replace(',', ' '));
                    }
                }
                DataSet ds = dbHelper.ExecuteQuery(sbsql.ToString());
                DataRow[] rows = ds.Tables[0].Select("Status='true'");
                if (rows.Length > 0)
                {
                    DataTable dtNew = ds.Tables[0].Clone();
                    for (int i = 0; i < rows.Length; i++)
                    {
                        dtNew.Rows.Add(rows[i].ItemArray);
                    }
                    return dtNew;
                }
                else
                {
                    return null;
                }
            }
            catch (Exception ex)
            {
                Log.Instance.Error("GetPointOnLines:" + ex.Message);
                return null;
            }
        }
 
        /// <summary>
        /// 判断某个经纬度点是否在这个区域内
        /// </summary>
        /// <param name="lonlat">经纬度点</param>
        /// <param name="area">多个经纬度点组成的区域集合，区域点必须是闭合的，即以哪个经纬度点开始必须以那个经纬度点结束</param>
        /// <returns>ture/false/null</returns>
        public DataTable GetPointOnTheArea(string lonlat, List<string> area)
        {
            try
            {
                StringBuilder sbarea = new StringBuilder();
                for (int i = 0; i < area.Count; i++)
                {
                    sbarea.Append(area[i].Replace(',', ' ') + ",");
                }
                string strsql = string.Format(@"SELECT ST_Contains( ST_MakePolygon(ST_GeomFromText('LINESTRING({0}) '))
                    ,st_point({1})) AS Status", sbarea.ToString().Trim(','), lonlat);
                return dbHelper.ExecuteQuery(strsql).Tables[0];
            }
            catch (Exception ex)
            {
                Log.Instance.Error("GetPointOnTheArea:" + ex.Message);
                return null;
            }
        }
 
        /// <summary>
        /// 判断某个经纬度点所在区域
        /// </summary>
        /// <param name="lonlat">经纬度点</param>
        /// <param name="areas">多个区域的经纬度集合</param>
        /// <returns>区域索引集合/null</returns>
        public DataTable GetPointOnAreas(string lonlat, List<List<string>> areas)
        {
            try
            {
                StringBuilder sbsql = new StringBuilder();
                for (int i = 0; i < areas.Count; i++)
                {
                    StringBuilder sbarea = new StringBuilder();
                    for (int j = 0; j < areas[i].Count; j++)
                    {
                        sbarea.Append(areas[i][j].Replace(',', ' ') + ",");
                    }
                    if (i == 0)
                    {
                        sbsql.AppendFormat(@"SELECT {0} AS Index, ST_Contains( ST_MakePolygon(ST_GeomFromText('LINESTRING({1}) '))
                            ,st_point({2})) AS Status", i, sbarea.ToString().Trim(','), lonlat);
                    }
                    else
                    {
                        sbsql.AppendFormat(@" union all SELECT {0} AS Index, ST_Contains( ST_MakePolygon(ST_GeomFromText('LINESTRING({1}) '))
                            ,st_point({2})) AS Status", i, sbarea.ToString().Trim(','), lonlat);
                    }
                }
                DataSet ds = dbHelper.ExecuteQuery(sbsql.ToString());
                DataRow[] rows = ds.Tables[0].Select("Status='true'");
                if (rows.Length > 0)
                {
                    DataTable dtNew = ds.Tables[0].Clone();
                    for (int i = 0; i < rows.Length; i++)
                    {
                        dtNew.Rows.Add(rows[i].ItemArray);
                    }
                    return dtNew;
                }
                else
                {
                    return null;
                }
            }
            catch (Exception ex)
            {
                Log.Instance.Error("GetPointOnOneArea:" + ex.Message);
                return null;
            }
        }
 
        /// <summary>
        /// 获取路线长度
        /// </summary>
        /// <param name="line">路线经纬度点集合</param>
        /// <returns>路线长度（单位m）/null</returns>
        public DataTable GetLineLength(List<string> line)
        {
            try
            {
                StringBuilder sbline = new StringBuilder();
                for (int i = 0; i < line.Count; i++)
                {
                    sbline.Append(line[i].Replace(',', ' ') + ",");
                }
                string strsql = string.Format(@"select ST_Length(Geography(ST_GeomFromText('LINESTRING({0})'))) AS Length", sbline.ToString().Trim(','));
                return dbHelper.ExecuteQuery(strsql).Tables[0];
            }
            catch (Exception ex)
            {
                Log.Instance.Error("GetLineLength:" + ex.Message);
                return null;
            }
        }
    }
  ```

## 3. PostgreHelper
  ```
   public class PostgreHelper : IDBHelper
    {
        /// <summary>
        /// 读取数据返回DataSet
        /// </summary>
        /// <param name="sqrstr"></param>
        /// <returns></returns>
        public DataSet ExecuteQuery(string sqrstr)
        {
            try
            {
                using (NpgsqlConnection conn = new NpgsqlConnection(Config.PostgreConnectionString))
                {
                    using (NpgsqlDataAdapter da = new NpgsqlDataAdapter(sqrstr, conn))
                    {
                        DataSet ds = new DataSet();
                        da.Fill(ds, "ds");
                        return ds;
                    }
                }
            }
            catch (Exception ex)
            {
                Log.Instance.Error("ExecuteQuery:"+ex.Message);
                return null;
            }
        }
 
        /// <summary>
        /// 判断增删改执行状态
        /// </summary>
        /// <param name="sqrstr"></param>
        /// <returns></returns>
        public int ExecuteNonQuery(string sqrstr)
        {
            try
            {
                using (NpgsqlConnection conn = new NpgsqlConnection(Config.PostgreConnectionString))
                {
                    using (NpgsqlCommand SqlCommand = new NpgsqlCommand(sqrstr, conn))
                    {
                        int r = SqlCommand.ExecuteNonQuery();  //执行查询并返回受影响的行数
                        conn.Close();
                        return r; //r如果是>0操作成功！ 
                    }
                }
            }
            catch (Exception ex)
            {
                Log.Instance.Error("ExecuteNonQuery:"+ex.Message);
                return -1;
            }
        }
 
        /// <summary>
        /// 读取数据返回数据流
        /// </summary>
        /// <param name="cmdText"></param>
        /// <returns></returns>
        public DbDataReader GetReader(string cmdText)
        {
            try
            {
                using (NpgsqlConnection conn = new NpgsqlConnection(Config.PostgreConnectionString))
                {
                    using (NpgsqlCommand cmd = new NpgsqlCommand(cmdText, conn))
                    {
                        NpgsqlDataReader sdr = cmd.ExecuteReader(CommandBehavior.CloseConnection);
                        return sdr;
                    }
                }
            }
            catch (Exception ex)
            {
                Log.Instance.Error("GetReader:" + ex.Message);
                return null;
            }
        }
    }

  ```
  ## 2. PostgreSQL中的GIS数据类型
  
	PostgreSQL提供了几种不同的GIS数据类型，用于存储空间数据。这些数据类型包括：
	1) 点（Point）：表示地球表面的一个点，由经度和纬度坐标组成。
	2) 线（LineString）：表示连接两个或多个点的线。
	3) 多边形（Polygon）：表示一个由三个或以上点组成的封闭区域。
	4) 多点（MultiPoint）：表示多个点的集合。
	5) 多线（MultiLineString）：表示多条线的集合。
	6) 多多边形（MultiPolygon）：表示多个多边形的集合。
	7) 几何集合（GeometryCollection）：表示不同类型的几何对象的集合。
	在PostgreSQL中，可以使用“geometry”类型来存储这些GIS数据类型。例如，以下命令将创建一个包含“point”类型的
	几何图形的表：

	CREATE TABLE my_points (id SERIAL PRIMARY KEY, shape GEOMETRY(Point, 4326));
  
  ##  3. PostgreSQL中的GIS函数

	PostgreSQL提供了许多GIS函数，可以用来处理空间数据。这些函数包括：
	1) ST_Intersects：用于检查两个几何对象是否相交。
	2) ST_Contains：用于检查一个几何对象是否包含另一个几何对象。
	3) ST_Distance：返回两个几何对象之间的距离。
	4) ST_Area：计算多边形的面积。
	5) ST_Length：计算线的长度。
	6) ST_Centroid：计算多边形的重心。
	7) ST_Buffer：生成一个缓冲区。
	以下是一个示例查询，使用ST_Intersects函数查找包含在区域内的所有点：

	SELECT * FROM my_points WHERE ST_Intersects(shape, ST_MakeEnvelope(-90, 30, -80, 40, 4326));
 
  ## 4. PostgreSQL中的GIS扩展

  ```
  	PostgreSQL的GIS功能可以通过GIS扩展进一步扩展。这些扩展可以提供额外的GIS功能，例如2D和3D地图可视化。其
	中一些扩展包括：
	1) PostGIS：提供了一组GIS函数和类型，用于空间数据的存储、查询和分析。
	2) pgRouting：提供了一组GIS函数，用于执行路径搜索和路由分析。
	3) PostGIS Raster：允许用户在PostgreSQL中存储和查询栅格数据（例如卫星图像）。
	4) Pointcloud：允许用户在PostgreSQL中存储和查询大型点云数据。
	以下是一个示例查询，使用PostGIS扩展创建地图：

	SELECT ST_AsPNG(ST_AsRaster(ST_Buffer(shape, 1, ‘quad_segs=4’), 100, 100, ARRAY[‘8BUI’, ‘8BUI’, ‘8BUI’], ARRAY[0, 0, 255]));

	这个查询将生成一个以点坐标为中心的缓冲区，使用四个段落的几何图形，并将其转换为栅格图像。然后，它将图像
	转换为PNG格式，并返回结果。

  ```

# 152  Enable PostGIS

## 1. create database with postgreSQL
```
创建testdb数据库：

CREATE DATABASE testdb;
复制
3.1.3 复制数据库
创建demo数据库，内容与testdb数据库一致：

CREATE DATABASE demo TEMPLATE=testdb;
复制
3.1.4 删除数据库
删除demo数据库：

drop database demo;
复制
3.1.5 查看数据库列表
执行\l来查看数据库列表


CREATE TABLE location_city (
    name            varchar(80),
    location        point
);

INSERT INTO location_city VALUES ('San Francisco', '(-194.0, 53.0)'), ('New York', '(-184.0, 43.0)'), ('北京', '(-94.0, 133.0)'), ('Los Angeles', '(-297.0, 63.0)'), ('Chicago', '(-94.0, 283.0)');

SELECT * FROM public.location_city

UPDATE location_city SET location = '(52,53)' WHERE name = 'Fort Worth';

DELETE FROM location_city WHERE name = 'San Francisco';

DELETE FROM location_city;
# 或者
TRUNCATE location_city;
```

## 2. create database with postgis

```
CREATE EXTENSION postgis;
-- Enable Topology
CREATE EXTENSION postgis_topology;
-- Enable PostGIS Advanced 3D-- and other geoprocessing algorithms
-- sfcgal not available with all distributions
CREATE EXTENSION postgis_sfcgal;
-- fuzzy matching needed for Tiger
CREATE EXTENSION fuzzystrmatch;
-- rule based standardizer
CREATE EXTENSION address_standardizer;
-- example rule data set
CREATE EXTENSION address_standardizer_data_us;
-- Enable US Tiger Geocoder
CREATE EXTENSION postgis_tiger_geocoder;

CREATE TABLE cities(id smallint,name varchar(50));
INSERT INTO cities(id, the_geom, name) VALUES (1,ST_GeomFromText('POINT(-0.1257 51.508)',4326),'London, England'), (2,ST_GeomFromText('POINT(-81.233 42.983)',4326),'London, Ontario'), (3,ST_GeomFromText('POINT(27.91162491 -33.01529)',4326),'East London,SA');


SELECT id, ST_AsText(the_geom), ST_AsEwkt(the_geom), ST_X(the_geom), ST_Y(the_geom) FROM cities;

SELECT * FROM pg_available_extensions WHERE name = 'postgis';

```
### 2.1. PostGIS EXTENSION

```
	-- Enable PostGIS (as of 3.0 contains just geometry/geography)
	-- 启用PostGIS功能（仅包括geometry/geography相关）
	CREATE EXTENSION postgis;

	-- enable raster support (for 3+)
	-- 启用栅格扩展
	CREATE EXTENSION postgis_raster;

	-- Enable Topology
	-- 启用拓扑扩展
	CREATE EXTENSION postgis_topology;

	-- Enable PostGIS Advanced 3D
	-- and other geoprocessing algorithms
	-- sfcgal not available with all distributions
	CREATE EXTENSION postgis_sfcgal;
	-- fuzzy matching needed for Tiger
	CREATE EXTENSION fuzzystrmatch;
	-- rule based standardizer
	CREATE EXTENSION address_standardizer;
	-- example rule data set
	CREATE EXTENSION address_standardizer_data_us;
	-- Enable US Tiger Geocoder
	CREATE EXTENSION postgis_tiger_geocoder;



	启动postgresql服务，进入命令行操作界面，执行以下命令： 

	create extension postgis

	select * from pg_available_extensions where name like 'postgis%';

```


## 3. some  postgis data example

### 3.1  ST_GeographyFromText

```
例如，下面是洛杉矶（Los Angeles）和巴黎（Paris）的地理坐标：
• Los Angeles: POINT(-118.4079 33.9434)
• Paris: POINT(2.3490 48.8533)
使用标准的PostGIS笛卡尔平面坐标系空间函数ST_Distance(geometry, geometry)计算洛杉矶和
巴黎之间的距离。请注意，SRID 4326声明了地理空间参考系统。
SELECT ST_Distance(
ST_GeometryFromText('POINT(-118.4079 33.9434)', 4326), -- Los Angeles (LAX)
ST_GeometryFromText('POINT(2.5559 49.0083)', 4326) -- Paris (CDG)
);
121.898

空间参考4326的单位是度，所以我们的答案是121度。但是，这表示什么呢？
在地球球体上，1度对应的地球实际距离的大小是变化的。当远离赤道时，它会变得更小，当越接
近两极时，地球上的经线相互之间越来越接近。因此，121度的距离并不意味着什么，这是一个没
有意义的数字。
为了计算出真实的距离，我们不能把地理坐标近似的看成笛卡尔平面坐标，而应该把它们看成是球
坐标。我们必须把两点之间的距离作为球面上的真实路径来测量——大圆（大圆被定义为过球心
的平面和球面的交线）的一部分。
从1.5版开始，PostGIS通过地理（geography）数据类型提供此功能

测量应该使用geography而不是geometry类型。也就是说使用geography这种数据类
型时，PostGIS的内部计算是基于实际地球球体来计算的；而使用geometry这种数据类型时，
PostGIS的内部计算是基于平面来计算的。
让我们再次尝试测量洛杉矶和巴黎之间的距离，我们将使用ST_GeographyFromText(text)函
数，而不是ST_GeometryFromText(text)。
SELECT ST_Distance(
ST_GeographyFromText('POINT(-118.4079 33.9434)'), -- Los Angeles (LAX)
ST_GeographyFromText('POINT(2.5559 49.0083)') -- Paris (CDG)
);

9124665

得到一个大数字！所有geography计算的返回值都以米为单位，所以我们的答案是9124km。

```

### 3.2  ST_GeographyFromText
```
如果我们将LAX-CDG航班路线转换成一条线串，并利用
geography计算其到冰岛某个点的距离，我们可以得到正确的答案（以米为单位）。
SELECT ST_Distance(
ST_GeographyFromText('LINESTRING(-118.4079 33.9434, 2.5559 49.0083)'), -- LAX-CDG
ST_GeographyFromText('POINT(-22.6056 63.9850)') -- Iceland (K
);


SELECT ST_Distance(
ST_GeometryFromText('Point(-118.4079 33.9434)'), -- LAX
ST_GeometryFromText('Point(139.733 35.567)')) -- NRT (Tokyo/Narita)
AS geometry_distance,

SELECT ST_Distance(
ST_GeographyFromText('Point(-118.4079 33.9434)'), -- LAX
ST_GeographyFromText('Point(139.733 35.567)')) -- NRT (Tokyo/Narita)
AS geography_distance;


CREATE TABLE airports (
code VARCHAR(3),
geog GEOGRAPHY(Point)
);
INSERT INTO airports VALUES ('LAX', 'POINT(-118.4079 33.9434)');
INSERT INTO airports VALUES ('CDG', 'POINT(2.5559 49.0083)');
INSERT INTO airports VALUES ('KEF', 'POINT(-22.6056 63.9850)');
在表定义中，GEOGRAPHY(Point)将airport数据类型指定为点。新的geography字段不会在
geometry_columns视图中注册，相反，它们是在名为geography_columns的视图中注册的。
SELECT * FROM geography_columns;


```

### 3.3  ST_GeomFromText
```
	-- Create table with spatial column
	-- 创建一个表mytable，包含了一个空间列geom，格式是GEOMETRY(Point,26910)
	-- 这里的26910是一个SRID，稍后会做解释
	CREATE TABLE mytable (
	id SERIAL PRIMARY KEY,
	geom GEOMETRY(Point, 26910),
	name VARCHAR(128)
	);
	
	-- Add a spatial index
	-- 给geom列添加一个空间索引，索引类型为GIST
	CREATE INDEX mytable_gix
	ON mytable
	USING GIST (geom);
	
	-- Add a point
	-- 插入一个数据点
	INSERT INTO mytable (geom) VALUES (
	ST_GeomFromText('POINT(0 0)', 26910)
	);
	
	-- Query for nearby points
	-- 查询与给定点周围1000米的点
	SELECT id, name
	FROM mytable
	WHERE ST_DWithin(
	geom,
	ST_GeomFromText('POINT(0 0)', 26910),
	1000
	);

```

# 153  PostGIS version

## 1. QUESTION:
```
ERROR:  function postgis_full_version() does not exist
LINE 5: SELECT PostGIS_Full_Version();
               ^
HINT:  No function matches the given name and argument types. You might need to add explicit type casts.
SQL state: 42883
Character: 49

```
## 2. SOLUTION:
to fix this error, we should  create postgis extension firstly with the following sql command:

```
CREATE EXTENSION postgis;
```
after create postgis extension,in the target database,we get the table name 'spatial_ref_sys ';
to check the spatial_ref_sys table,we can use the following sql command:

```
SELECT * FROM public.spatial_ref_sys
```
## 3. Query version

```
SELECT PostGIS_PROJ_Version();
Rel. 5.2.0, September 15th, 2018

SELECT PostGIS_GEOS_Version();
3.8.0-CAPI-1.13.1 

SELECT PostGIS_Full_Version();
POSTGIS="3.0.0 r17983" [EXTENSION] PGSQL="120" GEOS="3.8.0-CAPI-1.13.1 " SFCGAL="1.3.2" PROJ="Rel. 5.2.0, September 15th, 2018" LIBXML="2.9.9" LIBJSON="0.12" LIBPROTOBUF="1.2.1" WAGYU="0.4.3 (Internal)" TOPOLOGY


```

```
-- Table: public.roads

-- DROP TABLE public.roads;

CREATE TABLE public.roads
(
    road_id integer NOT NULL DEFAULT nextval('mytable_id_seq'::regclass),
    roads_geom geometry(Point,26910),
    road_name character varying(128) COLLATE pg_catalog."default",
    CONSTRAINT roads_pkey PRIMARY KEY (road_id)
)

TABLESPACE pg_default;

ALTER TABLE public.roads
    OWNER to postgres;


```

# 154 PostGIS Shapefile Import/Export Manager

 from postgis.net, get the manager to import shp file into postgis or export the table to shp file;



# 155  drawRichText with QPaint

## 1. html string 


<body bgcolor="0" size="12" style="writing-mode:horizontal-tb;"><p><font face="0" style="color: rgb(0, 0, 239); font-size: 18pt; background-color: rgb(217, 255, 255);">标注</font></p></body>

```
"<body bgcolor=\"0\" size=\"12\" style=\"writing-mode:horizontal-tb;\"><p><font face=\"0\" style=\"color: rgb(0, 0, 239); font-size: 18pt; background-color: rgb(217, 255, 255);\">标注</font></p></body>"
```
K<sub>max</sub>=K<sub>2</sub> + K<sup>2</sup> + K<sup>n+n<sup>2</sup></sup> - 3<sup>K/2</sup>

```
K<sub>max</sub>=K<sub>2</sub> + K<sup>2</sup> + K<sup>n+n<sup>2</sup></sup> - 3<sup>K/2</sup>
```
 a<sup>n+1</sup>

```
上角标的代码为：<sup>上角</sup>。如：a的n次方 a<sup>n</sup>显示结果an
```
 C<sub>3</sub>
```
下角标的代码为：<sub>下角</sub>。如：C<sub>3</sub>显示结果C3
```

H<sub>2</sub>O
```
H<sub>2</sub>O
```

Na<sub>2</sub>CO<sub>3</sub>
```
Na<sub>2</sub>CO<sub>3</sub>
```



<body bgcolor="0" size="12" style="writing-mode:horizontal-tb;"><p><font face="0">标注</font></p></body>


```
"<body bgcolor=\"0\" size=\"12\" style=\"writing-mode:horizontal-tb;\"><p><font face=\"0\">标注</font></p></body>"
```

<body bgcolor="0" size="8" style="writing-mode:horizontal-tb;"><p><font face="0" style="color: rgb(227, 127, 127);font-size: 18pt;">标注2</font></p></body>

```
<body bgcolor="0" size="8" style="writing-mode:horizontal-tb;"><p><font face="0" style="color: rgb(227, 127, 127);font-size: 18pt;">标注2</font></p></body>
```


## 2. function
```
void drawRichText(double originX, double originY, const xString & richtext, double width,double dpi = 72)
{
	QPainter* pPainter = new QPainter;
	QPaintDevice* pDevice = new QImage();
	double dScale = pDisplay->GetResolution();
	double nHei = pDevice->Height();
	originY = nHei - originY;
	pPainter->Save();
	pPainter->translate(originX, originY);

	double xScale = (dScale / dpi * 25.4);
	double yScale = (dScale / dpi * 25.4);
	pPainter->scale(xScale, yScale);
	pPainter->drawRichText(0, 0, xString(richtext, xString::GB18030), width * dpi / 25.4);
	pPainter->Restore();
}


void CPaint::drawRichText(double x, double y, const xString & richtext, double dwid)
{
	// 修改说明：QPainter的drawstatictext方法对html语法支持较弱
	// 使用QTextDocument绘制html
	
 	QTextOption option;
 	option.setWrapMode(QTextOption::WrapAnywhere);//设置Html文本可在任何地方换行

	QTextDocument doc;
	doc.setDefaultTextOption(option);
	doc.setHtml(ToQString(richtext));
	doc.setTextWidth(dwid); // 若不调用此句，则水平方向的对齐设置会被忽略；
	doc.drawContents(m_d->get()->qt);
}

```


# 156  get schemaname from postgresql

```
SELECT * FROM information_schema.schemata

select * from pg_tables where schemaname = 'public'

select * from pg_tables where schemaname = 'postgres'

select * from pg_tables where schemaname = 'information_schema'

SELECT schemaname,tablename FROM pg_tables 
 WHERE schemaname NOT LIKE 'pg%'
AND schemaname NOT LIKE 'information_schema%' order by schemaname

```
```
/*
 Navicat Premium Data Transfer

 Source Server         : osm
 Source Server Type    : PostgreSQL
 Source Server Version : 120001
 Source Host           : localhost:5432
 Source Catalog        : osm
 Source Schema         : dbmaster

 Target Server Type    : PostgreSQL
 Target Server Version : 120001
 File Encoding         : 65001

 Date: 14/08/2023 20:28:21
*/

*/
-- ----------------------------
-- Table structure for gdbinfo
-- ----------------------------
DROP TABLE IF EXISTS "dbmaster"."gdbinfo";
CREATE TABLE "dbmaster"."gdbinfo" (
  "id" int4 NOT NULL,
  "owner" varchar(128) COLLATE "pg_catalog"."default" NOT NULL,
  "name" varchar(128) COLLATE "pg_catalog"."default" NOT NULL,
  "status" int2 DEFAULT 0,
  "status2" int2 DEFAULT 0,
  "uuid" varchar(128) COLLATE "pg_catalog"."default" NOT NULL,
  "db_name" text COLLATE "pg_catalog"."default",
  "crttime" timestamp(6) NOT NULL DEFAULT CURRENT_TIMESTAMP,
  "backup_time" timestamp(6) NOT NULL DEFAULT CURRENT_TIMESTAMP
)
;

-- ----------------------------
-- Primary Key structure for table gdbinfo
-- ----------------------------
ALTER TABLE "dbmaster"."gdbinfo" ADD CONSTRAINT "gdbinfo_pkey" PRIMARY KEY ("id");
```

# 157  get column_name data_type from oracle

```
select column_name,data_type,data_length,data_precision,data_scale,nullable from all_tab_columns where table_name = 'SDEL' and owner = 'SDE' order by column_id

select a.SHAPE.SRID from SDE.SDER a

select a.FALSEX, a.FALSEY, a.xyunits, a.XYCLUSTER_TOL from sde.SPATIAL_REFERENCES a where a.auth_srid = 300015

SELECT OBJECTID,  a.SHAPE.points  ,  a.SHAPE.numpts ,a.SHAPE.SRID from SDE.SDER a  order by OBJECTID asc 

SELECT length(a.shape.points)  FROM SDE.SDER a 

SELECT max(length(a.shape.points))  FROM SDE.SDER a 

```

```
/*
 Navicat Premium Data Transfer

 Source Server         : 8185sde-land
 Source Server Type    : Oracle
 Source Server Version : 120200
 Source Host           : 192.168.81.85:1521
 Source Schema         : SDE

 Target Server Type    : Oracle
 Target Server Version : 120200
 File Encoding         : 65001

 Date: 22/08/2023 15:35:58
*/


-- ----------------------------
-- Table structure for SDER
-- ----------------------------
DROP TABLE "SDE"."SDER";
CREATE TABLE "SDE"."SDER" (
  "OBJECTID" NUMBER VISIBLE NOT NULL,
  "MPLAYER" NUMBER(9,0) VISIBLE,
  "MPAREA" NUMBER(38,8) VISIBLE,
  "MPPERIMETE" NUMBER(38,8) VISIBLE,
  "FLD_STR" NVARCHAR2(10) VISIBLE,
  "FLD_CHAR" NUMBER(3,0) VISIBLE,
  "FLD_BOOL" NUMBER(5,0) VISIBLE,
  "FLD_SHORT" NUMBER(4,0) VISIBLE,
  "FLD_LONG" NUMBER(9,0) VISIBLE,
  "FLD_INT64" NVARCHAR2(20) VISIBLE,
  "FLD_FLOAT" NUMBER(10,3) VISIBLE,
  "FLD_DOUBLE" NUMBER(38,8) VISIBLE,
  "FLD_DATE" TIMESTAMP(6) VISIBLE,
  "FLD_TIMSTA" TIMESTAMP(6) VISIBLE,
  "中文字段" NVARCHAR2(20) VISIBLE,
  "SHAPE" "SDE"."ST_GEOMETRY" VISIBLE
)
LOGGING
NOCOMPRESS

INITRANS 4
STORAGE (
  INITIAL 65536 
  NEXT 1048576 
  MINEXTENTS 1
  MAXEXTENTS 2147483645
  BUFFER_POOL DEFAULT
)
PARALLEL 1
NOCACHE
DISABLE ROW MOVEMENT
;

-- ----------------------------
-- Records of SDER
-- ----------------------------
INSERT INTO "SDE"."SDER" VALUES ('1', '0', '4299.276678', '268.504026', '嗡嗡嗡额外', '0', '1', '-327', '-21474836', '-9223372036854775807', '-9999.99', '-99999999.99999', TO_TIMESTAMP('2015-11-09 00:00:00.000000', 'SYYYY-MM-DD HH24:MI:SS:FF6'), TO_TIMESTAMP('2015-11-09 00:00:00.000000', 'SYYYY-MM-DD HH24:MI:SS:FF6'), '测试1', '"SDE"."ST_GEOMETRY"(8,5,-298.894260774326,172.380519692266,-214.416769336675,234.984374954098,NULL,NULL,NULL,NULL,4299.27667816376,268.504025829561,300015,HEXTORAW(''FF 1 0 0 1 0 0 0FFFFFFFFFFFFFF 3FFFFFFFFFFFFFF18FFFFFFFFFFFFFF 2FFFFFFFFFFFFFF 7FFFFFFFFFFFFFF BFFFFFFFFFFFF61FFFFFFFFFFFF1AFFFFFFFFFFFFFF 7FFFFFFFFFFFFFF 8FFFFFFFFFFFF3A''))');
INSERT INTO "SDE"."SDER" VALUES ('2', '0', '5017.485941', '252.478215', '为2332131', '127', '0', '3276', '214748364', '9223372036854775807', '9999.999', '99999999.999999', TO_TIMESTAMP('2015-11-09 00:00:00.000000', 'SYYYY-MM-DD HH24:MI:SS:FF6'), TO_TIMESTAMP('2015-11-09 00:00:00.000000', 'SYYYY-MM-DD HH24:MI:SS:FF6'), '我们', '"SDE"."ST_GEOMETRY"(8,2630,-200.813162398751,141.361707565162,-123.389955814072,225.206978551452,NULL,NULL,NULL,NULL,5017.48594120895,252.478214646624,300015,HEXTORAW(''FFFF 4 0 1 0 0 0FFFFFFFFFFFFFF10FFFFFFFFFFFFFF DFFFFFFFFFF36FFFFFFFFFF1EFFFFFFFFFFFF 1FFFFFFFFFF6BFFFFFFFFFFFF 1FFFFFFFFFF6AFFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF68FFFFFFFFFFFF 1FFFFFFFFFF67FFFFFFFFFFFF 1FFFFFFFFFF67FFFFFFFFFFFF 1FFFFFFFFFF66FFFFFFFFFFFF 1FFFFFFFFFF65FFFFFFFFFFFF 1FFFFFFFFFF64FFFFFFFFFFFF 1FFFFFFFFFF64FFFFFFFFFFFF 1FFFFFFFFFF63FFFFFFFFFFFF 1FFFFFFFFFF62FFFFFFFFFFFF 1FFFFFFFFFF61FFFFFFFFFFFF 1FFFFFFFFFF61FFFFFFFFFFFF 1FFFFFFFFFF60FFFFFFFFFFFF 1FFFFFFFFFF5FFFFFFFFFFFFF 1FFFFFFFFFF5FFFFFFFFFFFFF 1FFFFFFFFFF5EFFFFFFFFFFFF 1FFFFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF5CFFFFFFFFFFFF 1FFFFFFFFFF5BFFFFFFFFFFFF 1FFFFFFFFFF5AFFFFFFFFFFFF 1FFFFFFFFFF5AFFFFFFFFFFFF 1FFFFFFFFFF59FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF57FFFFFFFFFFFF 1FFFFFFFFFF56FFFFFFFFFFFF 1FFFFFFFFFF55FFFFFFFFFFFF 1FFFFFFFFFF55FFFFFFFFFFFF 1FFFFFFFFFF54FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF52FFFFFFFFFFFF 1FFFFFFFFFF51FFFFFFFFFFFF 1FFFFFFFFFF51FFFFFFFFFFFF 1FFFFFFFFFF50FFFFFFFFFFFF 1FFFFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF4EFFFFFFFFFFFF 1FFFFFFFFFF4EFFFFFFFFFFFF 1FFFFFFFFFF4DFFFFFFFFFFFF 1FFFFFFFFFF4CFFFFFFFFFFFF 1FFFFFFFFFF4CFFFFFFFFFFFF 1FFFFFFFFFF4BFFFFFFFFFFFF 1FFFFFFFFFF4AFFFFFFFFFFFF 1FFFFFFFFFF4AFFFFFFFFFFFF 1FFFFFFFFFF49FFFFFFFFFFFF 1FFFFFFFFFF48FFFFFFFFFFFF 1FFFFFFFFFF48FFFFFFFFFFFF 1FFFFFFFFFF47FFFFFFFFFFFF 1FFFFFFFFFF46FFFFFFFFFFFF 1FFFFFFFFFF46FFFFFFFFFFFF 1FFFFFFFFFF45FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF43FFFFFFFFFFFF 1FFFFFFFFFF42FFFFFFFFFFFF 1FFFFFFFFFF41FFFFFFFFFFFF 1FFFFFFFFFF41FFFFFFFFFFFF 1FFFFFFFFFF40FFFFFFFFFFFF 1FFFFFFFFFF3FFFFFFFFFFFFF 1FFFFFFFFFF3FFFFFFFFFFFFF 1FFFFFFFFFF3EFFFFFFFFFFFF 1FFFFFFFFFF3DFFFFFFFFFFFF 1FFFFFFFFFF3DFFFFFFFFFFFF 1FFFFFFFFFF3CFFFFFFFFFFFF 1FFFFFFFFFF3BFFFFFFFFFFFF 1FFFFFFFFFF3BFFFFFFFFFFFF 1FFFFFFFFFF3AFFFFFFFFFFFF 1FFFFFFFFFF39FFFFFFFFFFFF 1FFFFFFFFFF39FFFFFFFFFFFF 1FFFFFFFFFF38FFFFFFFFFFFF 1FFFFFFFFFF37FFFFFFFFFFFF 1FFFFFFFFFF37FFFFFFFFFFFF 1FFFFFFFFFF36FFFFFFFFFFFF 1FFFFFFFFFF35FFFFFFFFFFFF 1FFFFFFFFFF35FFFFFFFFFFFF 1FFFFFFFFFF34FFFFFFFFFFFF 1FFFFFFFFFF33FFFFFFFFFFFF 1FFFFFFFFFF33FFFFFFFFFFFF 1FFFFFFFFFF32FFFFFFFFFFFF 1FFFFFFFFFF31FFFFFFFFFFFF 1FFFFFFFFFF31FFFFFFFFFFFF 1FFFFFFFFFF30FFFFFFFFFFFF 1FFFFFFFFFF30FFFFFFFFFFFF 1FFFFFFFFFF2FFFFFFFFFFFFF 1FFFFFFFFFF2EFFFFFFFFFFFF 1FFFFFFFFFF2EFFFFFFFFFFFF 1FFFFFFFFFF2DFFFFFFFFFFFF 1FFFFFFFFFF2CFFFFFFFFFFFF 1FFFFFFFFFF2CFFFFFFFFFFFF 1FFFFFFFFFF2BFFFFFFFFFFFF 1FFFFFFFFFF2AFFFFFFFFFFFF 1FFFFFFFFFF2AFFFFFFFFFFFF 1FFFFFFFFFF29FFFFFFFFFFFF 1FFFFFFFFFF28FFFFFFFFFFFF 1FFFFFFFFFF28FFFFFFFFFFFF 1FFFFFFFFFF27FFFFFFFFFFFF 1FFFFFFFFFF26FFFFFFFFFFFF 1FFFFFFFFFF26FFFFFFFFFFFF 1FFFFFFFFFF25FFFFFFFFFFFF 1FFFFFFFFFF25FFFFFFFFFFFF 1FFFFFFFFFF24FFFFFFFFFFFF 1FFFFFFFFFF23FFFFFFFFFFFF 1FFFFFFFFFF23FFFFFFFFFFFF 1FFFFFFFFFF22FFFFFFFFFFFF 1FFFFFFFFFF21FFFFFFFFFFFF 1FFFFFFFFFF21FFFFFFFFFFFF 1FFFFFFFFFF20FFFFFFFFFFFF 1FFFFFFFFFF1FFFFFFFFFFFFF 1FFFFFFFFFF1FFFFFFFFFFFFF 1FFFFFFFFFF1EFFFFFFFFFFFF 1FFFFFFFFFF1EFFFFFFFFFFFF 1FFFFFFFFFF1DFFFFFFFFFFFF 1FFFFFFFFFF1CFFFFFFFFFFFF 1FFFFFFFFFF1CFFFFFFFFFFFF 1FFFFFFFFFF1BFFFFFFFFFFFF 1FFFFFFFFFF1AFFFFFFFFFFFF 1FFFFFFFFFF1AFFFFFFFFFFFF 1FFFFFFFFFF19FFFFFFFFFFFF 1FFFFFFFFFF19FFFFFFFFFFFF 1FFFFFFFFFF18FFFFFFFFFFFF 1FFFFFFFFFF17FFFFFFFFFFFF 1FFFFFFFFFF17FFFFFFFFFFFF 1FFFFFFFFFF16FFFFFFFFFFFF 1FFFFFFFFFF15FFFFFFFFFFFF 1FFFFFFFFFF15FFFFFFFFFFFF 1FFFFFFFFFF14FFFFFFFFFFFF 1FFFFFFFFFF14FFFFFFFFFFFF 1FFFFFFFFFF13FFFFFFFFFFFF 1FFFFFFFFFF12FFFFFFFFFFFF 1FFFFFFFFFF12FFFFFFFFFFFF 1FFFFFFFFFF11FFFFFFFFFFFF 1FFFFFFFFFF11FFFFFFFFFFFF 1FFFFFFFFFF10FFFFFFFFFFFF 1FFFFFFFFFF FFFFFFFFFFFFF 1FFFFFFFFFF FFFFFFFFFFFFF 1FFFFFFFFFF EFFFFFFFFFFFF 1FFFFFFFFFF DFFFFFFFFFFFF 1FFFFFFFFFF DFFFFFFFFFFFF 1FFFFFFFFFF CFFFFFFFFFFFF 1FFFFFFFFFF CFFFFFFFFFFFF 1FFFFFFFFFF BFFFFFFFFFFFF 1FFFFFFFFFF AFFFFFFFFFFFF 1FFFFFFFFFF AFFFFFFFFFFFF 1FFFFFFFFFF 9FFFFFFFFFFFF 1FFFFFFFFFF 9FFFFFFFFFFFF 1FFFFFFFFFF 8FFFFFFFFFFFF 1FFFFFFFFFF 7FFFFFFFFFFFF 1FFFFFFFFFF 7FFFFFFFFFFFF 1FFFFFFFFFF 6FFFFFFFFFFFF 1FFFFFFFFFF 6FFFFFFFFFFFF 1FFFFFFFFFF 5FFFFFFFFFFFF 1FFFFFFFFFF 4FFFFFFFFFFFF 1FFFFFFFFFF 4FFFFFFFFFFFF 1FFFFFFFFFF 3FFFFFFFFFFFF 1FFFFFFFFFF 3FFFFFFFFFFFF 1FFFFFFFFFF 2FFFFFFFFFFFF 1FFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFF67FFFFFFFFFFFF 1FFFFFFFF1CFFFFFFFFFFFF 1FFFFFFFF2FFFFFFFFFFFFF 1FFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFF 2FFFFFFFFFFFF 1FFFFFFFFFF 2FFFFFFFFFFFF 1FFFFFFFFFF 3FFFFFFFFFFFF 1FFFFFFFFFF 3FFFFFFFFFFFF 1FFFFFFFFFF 4FFFFFFFFFFFF 1FFFFFFFFFF 5FFFFFFFFFFFF 1FFFFFFFFFF 5FFFFFFFFFFFF 1FFFFFFFFFF 6FFFFFFFFFFFF 1FFFFFFFFFF 6FFFFFFFFFFFF 1FFFFFFFFFF 7FFFFFFFFFFFF 1FFFFFFFFFF 7FFFFFFFFFFFF 1FFFFFFFFFF 8FFFFFFFFFFFF 1FFFFFFFFFF 9FFFFFFFFFFFF 1FFFFFFFFFF 9FFFFFFFFFFFF 1FFFFFFFFFF AFFFFFFFFFFFF 1FFFFFFFFFF AFFFFFFFFFFFF 1FFFFFFFFFF BFFFFFFFFFFFF 1FFFFFFFFFF BFFFFFFFFFFFF 1FFFFFFFFFF CFFFFFFFFFFFF 1FFFFFFFFFF DFFFFFFFFFFFF 1FFFFFFFFFF DFFFFFFFFFFFF 1FFFFFFFFFF EFFFFFFFFFFFF 1FFFFFFFFFF EFFFFFFFFFFFF 1FFFFFFFFFF FFFFFFFFFFFFF 1FFFFFFFFFF FFFFFFFFFFFFF 1FFFFFFFFFF10FFFFFFFFFFFF 1FFFFFFFFFF11FFFFFFFFFFFF 1FFFFFFFFFF11FFFFFFFFFFFF 1FFFFFFFFFF12FFFFFFFFFFFF 1FFFFFFFFFF12FFFFFFFFFFFF 1FFFFFFFFFF13FFFFFFFFFFFF 1FFFFFFFFFF13FFFFFFFFFFFF 1FFFFFFFFFF14FFFFFFFFFFFF 1FFFFFFFFFF14FFFFFFFFFFFF 1FFFFFFFFFF15FFFFFFFFFFFF 1FFFFFFFFFF16FFFFFFFFFFFF 1FFFFFFFFFF16FFFFFFFFFFFF 1FFFFFFFFFF17FFFFFFFFFFFF 1FFFFFFFFFF17FFFFFFFFFFFF 1FFFFFFFFFF18FFFFFFFFFFFF 1FFFFFFFFFF18FFFFFFFFFFFF 1FFFFFFFFFF19FFFFFFFFFFFF 1FFFFFFFFFF19FFFFFFFFFFFF 1FFFFFFFFFF1AFFFFFFFFFFFF 1FFFFFFFFFF1BFFFFFFFFFFFF 1FFFFFFFFFF1BFFFFFFFFFFFF 1FFFFFFFFFF1CFFFFFFFFFFFF 1FFFFFFFFFF1CFFFFFFFFFFFF 1FFFFFFFFFF1DFFFFFFFFFFFF 1FFFFFFFFFF1DFFFFFFFFFFFF 1FFFFFFFFFF1EFFFFFFFFFFFF 1FFFFFFFFFF1EFFFFFFFFFFFF 1FFFFFFFFFF1FFFFFFFFFFFFF 1FFFFFFFFFF1FFFFFFFFFFFFF 1FFFFFFFFFF20FFFFFFFFFFFF 1FFFFFFFFFF21FFFFFFFFFFFF 1FFFFFFFFFF21FFFFFFFFFFFF 1FFFFFFFFFF22FFFFFFFFFFFF 1FFFFFFFFFF22FFFFFFFFFFFF 1FFFFFFFFFF23FFFFFFFFFFFF 1FFFFFFFFFF23FFFFFFFFFFFF 1FFFFFFFFFF24FFFFFFFFFFFF 1FFFFFFFFFF24FFFFFFFFFFFF 1FFFFFFFFFF25FFFFFFFFFFFF 1FFFFFFFFFF25FFFFFFFFFFFF 1FFFFFFFFFF26FFFFFFFFFFFF 1FFFFFFFFFF26FFFFFFFFFFFF 1FFFFFFFFFF27FFFFFFFFFFFF 1FFFFFFFFFF27FFFFFFFFFFFF 1FFFFFFFFFF28FFFFFFFFFFFF 1FFFFFFFFFF29FFFFFFFFFFFF 1FFFFFFFFFF29FFFFFFFFFFFF 1FFFFFFFFFF2AFFFFFFFFFFFF 1FFFFFFFFFF2AFFFFFFFFFFFF 1FFFFFFFFFF2BFFFFFFFFFFFF 1FFFFFFFFFF2BFFFFFFFFFFFF 1FFFFFFFFFF2CFFFFFFFFFFFF 1FFFFFFFFFF2CFFFFFFFFFFFF 1FFFFFFFFFF2DFFFFFFFFFFFF 1FFFFFFFFFF2DFFFFFFFFFFFF 1FFFFFFFFFF2EFFFFFFFFFFFF 1FFFFFFFFFF2EFFFFFFFFFFFF 1FFFFFFFFFF2FFFFFFFFFFFFF 1FFFFFFFFFF2FFFFFFFFFFFFF 1FFFFFFFFFF30FFFFFFFFFFFF 1FFFFFFFFFF30FFFFFFFFFFFF 1FFFFFFFFFF31FFFFFFFFFFFF 1FFFFFFFFFF31FFFFFFFFFFFF 1FFFFFFFFFF32FFFFFFFFFFFF 1FFFFFFFFFF32FFFFFFFFFFFF 1FFFFFFFFFF33FFFFFFFFFFFF 1FFFFFFFFFF33FFFFFFFFFFFF 1FFFFFFFFFF34FFFFFFFFFFFF 1FFFFFFFFFF34FFFFFFFFFFFF 1FFFFFFFFFF35FFFFFFFFFFFF 1FFFFFFFFFF35FFFFFFFFFFFF 1FFFFFFFFFF36FFFFFFFFFFFF 1FFFFFFFFFF36FFFFFFFFFFFF 1FFFFFFFFFF37FFFFFFFFFFFF 1FFFFFFFFFF37FFFFFFFFFFFF 1FFFFFFFFFF38FFFFFFFFFFFF 1FFFFFFFFFF38FFFFFFFFFFFF 1FFFFFFFFFF39FFFFFFFFFFFF 1FFFFFFFFFF39FFFFFFFFFFFF 1FFFFFFFFFF3AFFFFFFFFFFFF 1FFFFFFFFFF3AFFFFFFFFFFFF 1FFFFFFFFFF3BFFFFFFFFFFFF 1FFFFFFFFFF3BFFFFFFFFFFFF 1FFFFFFFFFF3CFFFFFFFFFFFF 1FFFFFFFFFF3CFFFFFFFFFFFF 1FFFFFFFFFF3DFFFFFFFFFFFF 1FFFFFFFFFF3DFFFFFFFFFFFF 1FFFFFFFFFF3EFFFFFFFFFFFF 1FFFFFFFFFF3EFFFFFFFFFFFF 1FFFFFFFFFF3FFFFFFFFFFFFF 1FFFFFFFFFF3FFFFFFFFFFFFF 1FFFFFFFFFF40FFFFFFFFFFFF 1FFFFFFFFFF40FFFFFFFFFFFF 1FFFFFFFFFF41FFFFFFFFFFFF 1FFFFFFFFFF41FFFFFFFFFFFF 1FFFFFFFFFF42FFFFFFFFFFFF 1FFFFFFFFFF42FFFFFFFFFFFF 1FFFFFFFFFF43FFFFFFFFFFFF 1FFFFFFFFFF43FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF45FFFFFFFFFFFF 1FFFFFFFFFF45FFFFFFFFFFFF 1FFFFFFFFFF46FFFFFFFFFFFF 1FFFFFFFFFF46FFFFFFFFFFFF 1FFFFFFFFFF47FFFFFFFFFFFF 1FFFFFFFFFF47FFFFFFFFFFFF 1FFFFFFFFFF47FFFFFFFFFFFF 1FFFFFFFFFF48FFFFFFFFFFFF 1FFFFFFFFFF48FFFFFFFFFFFF 1FFFFFFFFFF49FFFFFFFFFFFF 1FFFFFFFFFF49FFFFFFFFFFFF 1FFFFFFFFFF4AFFFFFFFFFFFF 1FFFFFFFFFF4AFFFFFFFFFFFF 1FFFFFFFFFF4BFFFFFFFFFFFF 1FFFFFFFFFF4BFFFFFFFFFFFF 1FFFFFFFFFF4CFFFFFFFFFFFF 1FFFFFFFFFF4CFFFFFFFFFFFF 1FFFFFFFFFF4DFFFFFFFFFFFF 1FFFFFFFFFF4DFFFFFFFFFFFF 1FFFFFFFFFF4EFFFFFFFFFFFF 1FFFFFFFFFF4EFFFFFFFFFFFF 1FFFFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF50FFFFFFFFFFFF 1FFFFFFFFFF50FFFFFFFFFFFF 1FFFFFFFFFF51FFFFFFFFFFFF 1FFFFFFFFFF51FFFFFFFFFFFF 1FFFFFFFFFF52FFFFFFFFFFFF 1FFFFFFFFFF52FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF54FFFFFFFFFFFF 1FFFFFFFFFF54FFFFFFFFFFFF 1FFFFFFFFFF54FFFFFFFFFFFF 1FFFFFFFFFF55FFFFFFFFFFFF 1FFFFFFFFFF55FFFFFFFFFFFF 1FFFFFFFFFF56FFFFFFFFFFFF 1FFFFFFFFFF56FFFFFFFFFFFF 1FFFFFFFFFF57FFFFFFFFFFFF 1FFFFFFFFFF57FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF59FFFFFFFFFFFF 1FFFFFFFFFF59FFFFFFFFFFFF 1FFFFFFFFFF5AFFFFFFFFFFFF 1FFFFFFFFFF5AFFFFFFFFFFFF 1FFFFFFFFFF5BFFFFFFFFFFFF 1FFFFFFFFFF5BFFFFFFFFFFFF 1FFFFFFFFFF5CFFFFFFFFFFFF 1FFFFFFFFFF5CFFFFFFFFFFFF 1FFFFFFFFFF5CFFFFFFFFFFFF 1FFFFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF5EFFFFFFFFFFFF 1FFFFFFFFFF5EFFFFFFFFFFFF 1FFFFFFFFFF5FFFFFFFFFFFFF 1FFFFFFFFFF5FFFFFFFFFFFFF 1FFFFFFFFFF5FFFFFFFFFFFFF 1FFFFFFFFFF60FFFFFFFFFFFF 1FFFFFFFFFF60FFFFFFFFFFFF 1FFFFFFFFFF61FFFFFFFFFFFF 1FFFFFFFFFF61FFFFFFFFFFFF 1FFFFFFFFFF62FFFFFFFFFFFF 1FFFFFFFFFF62FFFFFFFFFFFF 1FFFFFFFFFF62FFFFFFFFFFFF 1FFFFFFFFFF63FFFFFFFFFFFF 1FFFFFFFFFF63FFFFFFFFFFFF 1FFFFFFFFFF64FFFFFFFFFFFF 1FFFFFFFFFF64FFFFFFFFFFFF 1FFFFFFFFFF65FFFFFFFFFFFF 1FFFFFFFFFF65FFFFFFFFFFFF 1FFFFFFFFFF65FFFFFFFFFFFF 1FFFFFFFFFF66FFFFFFFFFFFF 1FFFFFFFFFF66FFFFFFFFFFFF 1FFFFFFFFFF67FFFFFFFFFFFF 1FFFFFFFFFF67FFFFFFFFFFFF 1FFFFFFFFFF67FFFFFFFFFFFF 1FFFFFFFFFF68FFFFFFFFFFFF 1FFFFFFFFFF68FFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF6AFFFFFFFFFFFF 1FFFFFFFFFF6AFFFFFFFFFFFF 1FFFFFFFFFF6BFFFFFFFFFFFF 1FFFFFFFFFF6BFFFFFFFFFFFF 1FFFFFFFFFF6BFFFFFFFFFFFF 1FFFFFFFFFF6CFFFFFFFFFFFF 1FFFFFFFFFF6CFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF70FFFFFFFFFFFF 1FFFFFFFFFF70FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF72FFFFFFFFFFFF 1FFFFFFFFFF72FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF77FFFFFFFFFFFF 1FFFFFFFFFF77FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF79FFFFFFFFFFFF 1FFFFFFFFFF79FFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7DFFFFFFFFFFFF 1FFFFFFFFFF7DFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7DFFFFFFFFFFFF 1FFFFFFFFFF7DFFFFFFFFFFFF 1FFFFFFFFFF7DFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF79FFFFFFFFFFFF 1FFFFFFFFFF79FFFFFFFFFFFF 1FFFFFFFFFF79FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF77FFFFFFFFFFFF 1FFFFFFFFFF77FFFFFFFFFFFF 1FFFFFFFFFF77FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF72FFFFFFFFFFFF 1FFFFFFFFFF72FFFFFFFFFFFF 1FFFFFFFFFF72FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF70FFFFFFFFFFFF 1FFFFFFFFFF70FFFFFFFFFFFF 1FFFFFFFFFF70FFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6CFFFFFFFFFFFF 1FFFFFFFFFF6CFFFFFFFFFFFF 1FFFFFFFFFF6BFFFFFFFFFFFF 1FFFFFFFFFF6BFFFFFFFFFFFF 1FFFFFFFFFF6BFFFFFFFFFFFF 1FFFFFFFFFF6AFFFFFFFFFFFF 1FFFFFFFFFF6AFFFFFFFFFFFF 1FFFFFFFFFF6AFFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF68FFFFFFFFFFFF 1FFFFFFFFFF68FFFFFFFFFFFF 1FFFFFFFFFF68FFFFFFFFFFFF 1FFFFFFFFFF67FFFFFFFFFFFF 1FFFFFFFFFF67FFFFFFFFFFFF 1FFFFFFFFFF66FFFFFFFFFFFF 1FFFFFFFFFF66FFFFFFFFFFFF 1FFFFFFFFFF66FFFFFFFFFFFF 1FFFFFFFFFF65FFFFFFFFFFFF 1FFFFFFFFFF65FFFFFFFFFFFF 1FFFFFFFFFF65FFFFFFFFFFFF 1FFFFFFFFFF64FFFFFFFFFFFF 1FFFFFFFFFF64FFFFFFFFFFFF 1FFFFFFFFFF63FFFFFFFFFFFF 1FFFFFFFFFF63FFFFFFFFFFFF 1FFFFFFFFFF63FFFFFFFFFFFF 1FFFFFFFFFF62FFFFFFFFFFFF 1FFFFFFFFFF62FFFFFFFFFFFF 1FFFFFFFFFF62FFFFFFFFFFFF 1FFFFFFFFFF61FFFFFFFFFFFF 1FFFFFFFFFF61FFFFFFFFFFFF 1FFFFFFFFFF60FFFFFFFFFFFF 1FFFFFFFFFF60FFFFFFFFFFFF 1FFFFFFFFFF60FFFFFFFFFFFF 1FFFFFFFFFF5FFFFFFFFFFFFF 1FFFFFFFFFF5FFFFFFFFFFFFF 1FFFFFFFFFF5FFFFFFFFFFFFF 1FFFFFFFFFF5EFFFFFFFFFFFF 1FFFFFFFFFF5EFFFFFFFFFFFF 1FFFFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF5CFFFFFFFFFFFF 1FFFFFFFFFF5CFFFFFFFFFFFF 1FFFFFFFFFF5CFFFFFFFFFFFF 1FFFFFFFFFF5BFFFFFFFFFFFF 1FFFFFFFFFF5BFFFFFFFFFFFF 1FFFFFFFFFF5AFFFFFFFFFFFF 1FFFFFFFFFF5AFFFFFFFFFFFF 1FFFFFFFFFF5AFFFFFFFFFFFF 1FFFFFFFFFF59FFFFFFFFFFFF 1FFFFFFFFFF59FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF57FFFFFFFFFFFF 1FFFFFFFFFF57FFFFFFFFFFFF 1FFFFFFFFFF56FFFFFFFFFFFF 1FFFFFFFFFF56FFFFFFFFFFFF 1FFFFFFFFFF56FFFFFFFFFFFF 1FFFFFFFFFF55FFFFFFFFFFFF 1FFFFFFFFFF55FFFFFFFFFFFF 1FFFFFFFFFF54FFFFFFFFFFFF 1FFFFFFFFFF54FFFFFFFFFFFF 1FFFFFFFFFF54FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF52FFFFFFFFFFFF 1FFFFFFFFFF52FFFFFFFFFFFF 1FFFFFFFFFF51FFFFFFFFFFFF 1FFFFFFFFFF51FFFFFFFFFFFF 1FFFFFFFFFF50FFFFFFFFFFFF 1FFFFFFFFFF50FFFFFFFFFFFF 1FFFFFFFFFF50FFFFFFFFFFFF 1FFFFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF4EFFFFFFFFFFFF 1FFFFFFFFFF4EFFFFFFFFFFFF 1FFFFFFFFFF4EFFFFFFFFFFFF 1FFFFFFFFFF4DFFFFFFFFFFFF 1FFFFFFFFFF4DFFFFFFFFFFFF 1FFFFFFFFFF4CFFFFFFFFFFFF 1FFFFFFFFFF4CFFFFFFFFFFFF 1FFFFFFFFFF4CFFFFFFFFFFFF 1FFFFFFFFFF4BFFFFFFFFFFFF 1FFFFFFFFFF4BFFFFFFFFFFFF 1FFFFFFFFFF4AFFFFFFFFFFFF 1FFFFFFFFFF4AFFFFFFFFFFFF 1FFFFFFFFFF4AFFFFFFFFFFFF 1FFFFFFFFFF49FFFFFFFFFFFF 1FFFFFFFFFF49FFFFFFFFFFFF 1FFFFFFFFFF48FFFFFFFFFFFF 1FFFFFFFFFF48FFFFFFFFFFFF 1FFFFFFFFFF47FFFFFFFFFFFF 1FFFFFFFFFF47FFFFFFFFFFFF 1FFFFFFFFFF47FFFFFFFFFFFF 1FFFFFFFFFF46FFFFFFFFFFFF 1FFFFFFFFFF46FFFFFFFFFFFF 1FFFFFFFFFF45FFFFFFFFFFFF 1FFFFFFFFFF45FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF43FFFFFFFFFFFF 1FFFFFFFFFF43FFFFFFFFFFFF 1FFFFFFFFFF42FFFFFFFFFFFF 1FFFFFFFFFF42FFFFFFFFFFFF 1FFFFFFFFFF42FFFFFFFFFFFF 1FFFFFFFFFF41FFFFFFFFFFFF 1FFFFFFFFFF41FFFFFFFFFFFF 1FFFFFFFFFF40FFFFFFFFFFFF 1FFFFFFFFFF40FFFFFFFFFFFF 1FFFFFFFFFF3FFFFFFFFFFFFF 1FFFFFFFFFF3FFFFFFFFFFFFF 1FFFFFFFFFF3EFFFFFFFFFFFF 1FFFFFFFFFF3EFFFFFFFFFFFF 1FFFFFFFFFF3EFFFFFFFFFFFF 1FFFFFFFFFF3DFFFFFFFFFFFF 1FFFFFFFFFF3DFFFFFFFFFFFF 1FFFFFFFFFF3CFFFFFFFFFFFF 1FFFFFFFFFF3CFFFFFFFFFFFF 1FFFFFFFFFF3BFFFFFFFFFFFF 1FFFFFFFFFF3BFFFFFFFFFFFF 1FFFFFFFFFF3BFFFFFFFFFFFF 1FFFFFFFFFF3AFFFFFFFFFFFF 1FFFFFFFFFF3AFFFFFFFFFFFF 1FFFFFFFFFF39FFFFFFFFFFFF 1FFFFFFFFFF39FFFFFFFFFFFF 1FFFFFFFFFF38FFFFFFFFFFFF 1FFFFFFFFFF38FFFFFFFFFFFF 1FFFFFFFFFF37FFFFFFFFFFFF 1FFFFFFFFFF37FFFFFFFFFFFF 1FFFFFFFFFF37FFFFFFFFFFFF 1FFFFFFFFFF36FFFFFFFFFFFF 1FFFFFFFFFF36FFFFFFFFFFFF 1FFFFFFFFFF35FFFFFFFFFFFF 1FFFFFFFFFF35FFFFFFFFFFFF 1FFFFFFFFFF34FFFFFFFFFFFF 1FFFFFFFFFF34FFFFFFFFFFFF 1FFFFFFFFFF33FFFFFFFFFFFF 1FFFFFFFFFF33FFFFFFFFFFFF 1FFFFFFFFFF33FFFFFFFFFFFF 1FFFFFFFFFF32FFFFFFFFFFFF 1FFFFFFFFFF32FFFFFFFFFFFF 1FFFFFFFFFF31FFFFFFFFFFFF 1FFFFFFFFFF31FFFFFFFFFFFF 1FFFFFFFFFF30FFFFFFFFFFFF 1FFFFFFFFFF30FFFFFFFFFFFF 1FFFFFFFFFF2FFFFFFFFFFFFF 1FFFFFFFFFF2FFFFFFFFFFFFF 1FFFFFFFFFF2EFFFFFFFFFFFF 1FFFFFFFFFF2EFFFFFFFFFFFF 1FFFFFFFFFF2EFFFFFFFFFFFF 1FFFFFFFFFF2DFFFFFFFFFFFF 1FFFFFFFFFF2DFFFFFFFFFFFF 1FFFFFFFFFF2CFFFFFFFFFFFF 1FFFFFFFFFF2CFFFFFFFFFFFF 1FFFFFFFFFF2BFFFFFFFFFFFF 1FFFFFFFFFF2BFFFFFFFFFFFF 1FFFFFFFFFF2AFFFFFFFFFFFF 1FFFFFFFFFF2AFFFFFFFFFFFF 1FFFFFFFFFF29FFFFFFFFFFFF 1FFFFFFFFFF29FFFFFFFFFFFF 1FFFFFFFFFF28FFFFFFFFFFFF 1FFFFFFFFFF28FFFFFFFFFFFF 1FFFFFFFFFF28FFFFFFFFFFFF 1FFFFFFFFFF27FFFFFFFFFFFF 1FFFFFFFFFF27FFFFFFFFFFFF 1FFFFFFFFFF26FFFFFFFFFFFF 1FFFFFFFFFF26FFFFFFFFFFFF 1FFFFFFFFFF25FFFFFFFFFFFF 1FFFFFFFFFF25FFFFFFFFFFFF 1FFFFFFFFFF24FFFFFFFFFFFF 1FFFFFFFFFF24FFFFFFFFFFFF 1FFFFFFFFFF23FFFFFFFFFFFF 1FFFFFFFFFF23FFFFFFFFFFFF 1FFFFFFFFFF22FFFFFFFFFFFF 1FFFFFFFFFF22FFFFFFFFFFFF 1FFFFFFFFFF21FFFFFFFFFFFF 1FFFFFFFFFF21FFFFFFFFFFFF 1FFFFFFFFFF20FFFFFFFFFFFF 1FFFFFFFFFF20FFFFFFFFFFFF 1FFFFFFFFFF1FFFFFFFFFFFFF 1FFFFFFFFFF1FFFFFFFFFFFFF 1FFFFFFFFFF1FFFFFFFFFFFFF 1FFFFFFFFFF1EFFFFFFFFFFFF 1FFFFFFFFFF1EFFFFFFFFFFFF 1FFFFFFFFFF1DFFFFFFFFFFFF 1FFFFFFFFFF1DFFFFFFFFFFFF 1FFFFFFFFFF1CFFFFFFFFFFFF 1FFFFFFFFFF1CFFFFFFFFFFFF 1FFFFFFFFFF1BFFFFFFFFFFFF 1FFFFFFFFFF1BFFFFFFFFFFFF 1FFFFFFFFFF1AFFFFFFFFFFFF 1FFFFFFFFFF1AFFFFFFFFFFFF 1FFFFFFFFFF19FFFFFFFFFFFF 1FFFFFFFFFF19FFFFFFFFFFFF 1FFFFFFFFFF18FFFFFFFFFFFF 1FFFFFFFFFF18FFFFFFFFFFFF 1FFFFFFFFFF17FFFFFFFFFFFF 1FFFFFFFFFF17FFFFFFFFFFFF 1FFFFFFFFFF16FFFFFFFFFFFF 1FFFFFFFFFF16FFFFFFFFFFFF 1FFFFFFFFFF15FFFFFFFFFFFF 1FFFFFFFFFF15FFFFFFFFFFFF 1FFFFFFFFFF14FFFFFFFFFFFF 1FFFFFFFFFF14FFFFFFFFFFFF 1FFFFFFFFFF13FFFFFFFFFFFF 1FFFFFFFFFF13FFFFFFFFFFFF 1FFFFFFFFFF12FFFFFFFFFFFF 1FFFFFFFFFF12FFFFFFFFFFFF 1FFFFFFFFFF11FFFFFFFFFFFF 1FFFFFFFFFF11FFFFFFFFFFFF 1FFFFFFFFFF10FFFFFFFFFFFF 1FFFFFFFFFF10FFFFFFFFFFFF 1FFFFFFFFFF FFFFFFFFFFFFF 1FFFFFFFFFF FFFFFFFFFFFFF 1FFFFFFFFFF EFFFFFFFFFFFF 1FFFFFFFFFF EFFFFFFFFFFFF 1FFFFFFFFFF DFFFFFFFFFFFF 1FFFFFFFFFF DFFFFFFFFFFFF 1FFFFFFFFFF CFFFFFFFFFFFF 1FFFFFFFFFF CFFFFFFFFFFFF 1FFFFFFFFFF BFFFFFFFFFFFF 1FFFFFFFFFF BFFFFFFFFFFFF 1FFFFFFFFFF AFFFFFFFFFFFF 1FFFFFFFFFF AFFFFFFFFFFFF 1FFFFFFFFFF 9FFFFFFFFFFFF 1FFFFFFFFFF 9FFFFFFFFFFFF 1FFFFFFFFFF 8FFFFFFFFFFFF 1FFFFFFFFFF 8FFFFFFFFFFFF 1FFFFFFFFFF 7FFFFFFFFFFFF 1FFFFFFFFFF 7FFFFFFFFFFFF 1FFFFFFFFFF 6FFFFFFFFFFFF 1FFFFFFFFFF 6FFFFFFFFFFFF 1FFFFFFFFFF 5FFFFFFFFFFFF 1FFFFFFFFFF 5FFFFFFFFFFFF 1FFFFFFFFFF 4FFFFFFFFFFFF 1FFFFFFFFFF 4FFFFFFFFFFFF 1FFFFFFFFFF 3FFFFFFFFFFFF 1FFFFFFFFFF 3FFFFFFFFFFFF 1FFFFFFFFFF 2FFFFFFFFFFFF 1FFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFF78FFFFFFFFFFFF 1FFFFFFFF36FFFFFFFFFFFF 1FFFFFFFF CFFFFFFFFFFFF 1FFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFF 2FFFFFFFFFFFF 1FFFFFFFFFF 2FFFFFFFFFFFF 1FFFFFFFFFF 3FFFFFFFFFFFF 1FFFFFFFFFF 3FFFFFFFFFFFF 1FFFFFFFFFF 4FFFFFFFFFFFF 1FFFFFFFFFF 4FFFFFFFFFFFF 1FFFFFFFFFF 5FFFFFFFFFFFF 1FFFFFFFFFF 5FFFFFFFFFFFF 1FFFFFFFFFF 6FFFFFFFFFFFF 1FFFFFFFFFF 6FFFFFFFFFFFF 1FFFFFFFFFF 7FFFFFFFFFFFF 1FFFFFFFFFF 8FFFFFFFFFFFF 1FFFFFFFFFF 8FFFFFFFFFFFF 1FFFFFFFFFF 9FFFFFFFFFFFF 1FFFFFFFFFF 9FFFFFFFFFFFF 1FFFFFFFFFF AFFFFFFFFFFFF 1FFFFFFFFFF AFFFFFFFFFFFF 1FFFFFFFFFF BFFFFFFFFFFFF 1FFFFFFFFFF BFFFFFFFFFFFF 1FFFFFFFFFF CFFFFFFFFFFFF 1FFFFFFFFFF CFFFFFFFFFFFF 1FFFFFFFFFF DFFFFFFFFFFFF 1FFFFFFFFFF DFFFFFFFFFFFF 1FFFFFFFFFF EFFFFFFFFFFFF 1FFFFFFFFFF FFFFFFFFFFFFF 1FFFFFFFFFF FFFFFFFFFFFFF 1FFFFFFFFFF10FFFFFFFFFFFF 1FFFFFFFFFF10FFFFFFFFFFFF 1FFFFFFFFFF11FFFFFFFFFFFF 1FFFFFFFFFF11FFFFFFFFFFFF 1FFFFFFFFFF12FFFFFFFFFFFF 1FFFFFFFFFF12FFFFFFFFFFFF 1FFFFFFFFFF13FFFFFFFFFFFF 1FFFFFFFFFF13FFFFFFFFFFFF 1FFFFFFFFFF14FFFFFFFFFFFF 1FFFFFFFFFF15FFFFFFFFFFFF 1FFFFFFFFFF15FFFFFFFFFFFF 1FFFFFFFFFF16FFFFFFFFFFFF 1FFFFFFFFFF16FFFFFFFFFFFF 1FFFFFFFFFF17FFFFFFFFFFFF 1FFFFFFFFFF17FFFFFFFFFFFF 1FFFFFFFFFF18FFFFFFFFFFFF 1FFFFFFFFFF18FFFFFFFFFFFF 1FFFFFFFFFF19FFFFFFFFFFFF 1FFFFFFFFFF19FFFFFFFFFFFF 1FFFFFFFFFF1AFFFFFFFFFFFF 1FFFFFFFFFF1BFFFFFFFFFFFF 1FFFFFFFFFF1BFFFFFFFFFFFF 1FFFFFFFFFF1CFFFFFFFFFFFF 1FFFFFFFFFF1CFFFFFFFFFFFF 1FFFFFFFFFF1DFFFFFFFFFFFF 1FFFFFFFFFF1DFFFFFFFFFFFF 1FFFFFFFFFF1EFFFFFFFFFFFF 1FFFFFFFFFF1FFFFFFFFFFFFF 1FFFFFFFFFF1FFFFFFFFFFFFF 1FFFFFFFFFF20FFFFFFFFFFFF 1FFFFFFFFFF20FFFFFFFFFFFF 1FFFFFFFFFF21FFFFFFFFFFFF 1FFFFFFFFFF21FFFFFFFFFFFF 1FFFFFFFFFF22FFFFFFFFFFFF 1FFFFFFFFFF22FFFFFFFFFFFF 1FFFFFFFFFF23FFFFFFFFFFFF 1FFFFFFFFFF24FFFFFFFFFFFF 1FFFFFFFFFF24FFFFFFFFFFFF 1FFFFFFFFFF25FFFFFFFFFFFF 1FFFFFFFFFF25FFFFFFFFFFFF 1FFFFFFFFFF26FFFFFFFFFFFF 1FFFFFFFFFF26FFFFFFFFFFFF 1FFFFFFFFFF27FFFFFFFFFFFF 1FFFFFFFFFF28FFFFFFFFFFFF 1FFFFFFFFFF28FFFFFFFFFFFF 1FFFFFFFFFF29FFFFFFFFFFFF 1FFFFFFFFFF29FFFFFFFFFFFF 1FFFFFFFFFF2AFFFFFFFFFFFF 1FFFFFFFFFF2AFFFFFFFFFFFF 1FFFFFFFFFF2BFFFFFFFFFFFF 1FFFFFFFFFF2CFFFFFFFFFFFF 1FFFFFFFFFF2CFFFFFFFFFFFF 1FFFFFFFFFF2DFFFFFFFFFFFF 1FFFFFFFFFF2DFFFFFFFFFFFF 1FFFFFFFFFF2EFFFFFFFFFFFF 1FFFFFFFFFF2EFFFFFFFFFFFF 1FFFFFFFFFF2FFFFFFFFFFFFF 1FFFFFFFFFF30FFFFFFFFFFFF 1FFFFFFFFFF30FFFFFFFFFFFF 1FFFFFFFFFF31FFFFFFFFFFFF 1FFFFFFFFFF31FFFFFFFFFFFF 1FFFFFFFFFF32FFFFFFFFFFFF 1FFFFFFFFFF33FFFFFFFFFFFF 1FFFFFFFFFF33FFFFFFFFFFFF 1FFFFFFFFFF34FFFFFFFFFFFF 1FFFFFFFFFF34FFFFFFFFFFFF 1FFFFFFFFFF35FFFFFFFFFFFF 1FFFFFFFFFF36FFFFFFFFFFFF 1FFFFFFFFFF36FFFFFFFFFFFF 1FFFFFFFFFF37FFFFFFFFFFFF 1FFFFFFFFFF37FFFFFFFFFFFF 1FFFFFFFFFF38FFFFFFFFFFFF 1FFFFFFFFFF38FFFFFFFFFFFF 1FFFFFFFFFF39FFFFFFFFFFFF 1FFFFFFFFFF3AFFFFFFFFFFFF 1FFFFFFFFFF3AFFFFFFFFFFFF 1FFFFFFFFFF3BFFFFFFFFFFFF 1FFFFFFFFFF3BFFFFFFFFFFFF 1FFFFFFFFFF3CFFFFFFFFFFFF 1FFFFFFFFFF3DFFFFFFFFFFFF 1FFFFFFFFFF3DFFFFFFFFFFFF 1FFFFFFFFFF3EFFFFFFFFFFFF 1FFFFFFFFFF3EFFFFFFFFFFFF 1FFFFFFFFFF3FFFFFFFFFFFFF 1FFFFFFFFFF40FFFFFFFFFFFF 1FFFFFFFFFF40FFFFFFFFFFFF 1FFFFFFFFFF41FFFFFFFFFFFF 1FFFFFFFFFF41FFFFFFFFFFFF 1FFFFFFFFFF42FFFFFFFFFFFF 1FFFFFFFFFF43FFFFFFFFFFFF 1FFFFFFFFFF43FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF45FFFFFFFFFFFF 1FFFFFFFFFF46FFFFFFFFFFFF 1FFFFFFFFFF46FFFFFFFFFFFF 1FFFFFFFFFF47FFFFFFFFFFFF 1FFFFFFFFFF47FFFFFFFFFFFF 1FFFFFFFFFF48FFFFFFFFFFFF 1FFFFFFFFFF49FFFFFFFFFFFF 1FFFFFFFFFF49FFFFFFFFFFFF 1FFFFFFFFFF4AFFFFFFFFFFFF 1FFFFFFFFFF4BFFFFFFFFFFFF 1FFFFFFFFFF4BFFFFFFFFFFFF 1FFFFFFFFFF4CFFFFFFFFFFFF 1FFFFFFFFFF4CFFFFFFFFFFFF 1FFFFFFFFFF4DFFFFFFFFFFFF 1FFFFFFFFFF4EFFFFFFFFFFFF 1FFFFFFFFFF4EFFFFFFFFFFFF 1FFFFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF50FFFFFFFFFFFF 1FFFFFFFFFF51FFFFFFFFFFFF 1FFFFFFFFFF51FFFFFFFFFFFF 1FFFFFFFFFF52FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF54FFFFFFFFFFFF 1FFFFFFFFFF54FFFFFFFFFFFF 1FFFFFFFFFF55FFFFFFFFFFFF 1FFFFFFFFFF56FFFFFFFFFFFF 1FFFFFFFFFF56FFFFFFFFFFFF 1FFFFFFFFFF57FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF59FFFFFFFFFFFF 1FFFFFFFFFF59FFFFFFFFFFFF 1FFFFFFFFFF5AFFFFFFFFFFFF 1FFFFFFFFFF5BFFFFFFFFFFFF 1FFFFFFFFFF5BFFFFFFFFFFFF 1FFFFFFFFFF5CFFFFFFFFFFFF 1FFFFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF5EFFFFFFFFFFFF 1FFFFFFFFFF5EFFFFFFFFFFFF 1FFFFFFFFFF5FFFFFFFFFFFFF 1FFFFFFFFFF60FFFFFFFFFFFF 1FFFFFFFFFF60FFFFFFFFFFFF 1FFFFFFFFFF61FFFFFFFFFFFF 1FFFFFFFFFF62FFFFFFFFFFFF 1FFFFFFFFFF62FFFFFFFFFFFF 1FFFFFFFFFF63FFFFFFFFFFFF 1FFFFFFFFFF64FFFFFFFFFFFF 1FFFFFFFFFF64FFFFFFFFFFFF 1FFFFFFFFFF65FFFFFFFFFFFF 1FFFFFFFFFF66FFFFFFFFFFFF 1FFFFFFFFFF66FFFFFFFFFFFF 1FFFFFFFFFF67FFFFFFFFFFFF 1FFFFFFFFFF67FFFFFFFFFFFF 1FFFFFFFFFF68FFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF6AFFFFFFFFFFFF 1FFFFFFFFFF6BFFFFFFFFFFFF 1FFFFFFFFFF6BFFFFFFFFFFFF 1FFFFFFFFFF6CFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF70FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF72FFFFFFFFFFFF 1FFFFFFFFFF72FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF77FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF79FFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7DFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF1DFFFFFFFFFF21FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7DFFFFFFFFFFFF 1FFFFFFFFFF7DFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF79FFFFFFFFFFFF 1FFFFFFFFFF79FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF77FFFFFFFFFFFF 1FFFFFFFFFF77FFFFFFFFFFFF 1FFFFFFFFFF77FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF72FFFFFFFFFFFF 1FFFFFFFFFF72FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF70FFFFFFFFFFFF 1FFFFFFFFFF70FFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6CFFFFFFFFFFFF 1FFFFFFFFFF6CFFFFFFFFFFFF 1FFFFFFFFFF6BFFFFFFFFFFFF 1FFFFFFFFFF6BFFFFFFFFFFFF 1FFFFFFFFFF6AFFFFFFFFFFFF 1FFFFFFFFFF6AFFFFFFFFFFFF 1FFFFFFFFFF6AFFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF68FFFFFFFFFFFF 1FFFFFFFFFF68FFFFFFFFFFFF 1FFFFFFFFFF67FFFFFFFFFFFF 1FFFFFFFFFF67FFFFFFFFFFFF 1FFFFFFFFFF66FFFFFFFFFFFF 1FFFFFFFFFF66FFFFFFFFFFFF 1FFFFFFFFFF66FFFFFFFFFFFF 1FFFFFFFFFF65FFFFFFFFFFFF 1FFFFFFFFFF65FFFFFFFFFFFF 1FFFFFFFFFF64FFFFFFFFFFFF 1FFFFFFFFFF64FFFFFFFFFFFF 1FFFFFFFFFF63FFFFFFFFFFFF 1FFFFFFFFFF63FFFFFFFFFFFF 1FFFFFFFFFF62FFFFFFFFFFFF 1FFFFFFFFFF62FFFFFFFFFFFF 1FFFFFFFFFF61FFFFFFFFFFFF 1FFFFFFFFFF61FFFFFFFFFFFF 1FFFFFFFFFF61FFFFFFFFFFFF 1FFFFFFFFFF60FFFFFFFFFFFF 1FFFFFFFFFF60FFFFFFFFFFFF 1FFFFFFFFFF5FFFFFFFFFFFFF 1FFFFFFFFFF5FFFFFFFFFFFFF 1FFFFFFFFFF5EFFFFFFFFFFFF 1FFFFFFFFFF5EFFFFFFFFFFFF 1FFFFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF5CFFFFFFFFFFFF 1FFFFFFFFFF5CFFFFFFFFFFFF 1FFFFFFFFFF5CFFFFFFFFFFFF 1FFFFFFFFFF5BFFFFFFFFFFFF 1FFFFFFFFFF5BFFFFFFFFFFFF 1FFFFFFFFFF5AFFFFFFFFFFFF 1FFFFFFFFFF5AFFFFFFFFFFFF 1FFFFFFFFFF59FFFFFFFFFFFF 1FFFFFFFFFF59FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF57FFFFFFFFFFFF 1FFFFFFFFFF57FFFFFFFFFFFF 1FFFFFFFFFF56FFFFFFFFFFFF 1FFFFFFFFFF56FFFFFFFFFFFF 1FFFFFFFFFF56FFFFFFFFFFFF 1FFFFFFFFFF55FFFFFFFFFFFF 1FFFFFFFFFF55FFFFFFFFFFFF 1FFFFFFFFFF54FFFFFFFFFFFF 1FFFFFFFFFF54FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF52FFFFFFFFFFFF 1FFFFFFFFFF52FFFFFFFFFFFF 1FFFFFFFFFF51FFFFFFFFFFFF 1FFFFFFFFFF51FFFFFFFFFFFF 1FFFFFFFFFF50FFFFFFFFFFFF 1FFFFFFFFFF50FFFFFFFFFFFF 1FFFFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF4EFFFFFFFFFFFF 1FFFFFFFFFF4EFFFFFFFFFFFF 1FFFFFFFFFF4DFFFFFFFFFFFF 1FFFFFFFFFF4DFFFFFFFFFFFF 1FFFFFFFFFF4CFFFFFFFFFFFF 1FFFFFFFFFF4CFFFFFFFFFFFF 1FFFFFFFFFF4BFFFFFFFFFFFF 1FFFFFFFFFF4BFFFFFFFFFFFF 1FFFFFFFFFF4AFFFFFFFFFFFF 1FFFFFFFFFF4AFFFFFFFFFFFF 1FFFFFFFFFF49FFFFFFFFFFFF 1FFFFFFFFFF49FFFFFFFFFFFF 1FFFFFFFFFF48FFFFFFFFFFFF 1FFFFFFFFFF48FFFFFFFFFFFF 1FFFFFFFFFF47FFFFFFFFFFFF 1FFFFFFFFFF47FFFFFFFFFFFF 1FFFFFFFFFF46FFFFFFFFFFFF 1FFFFFFFFFF46FFFFFFFFFFFF 1FFFFFFFFFF45FFFFFFFFFFFF 1FFFFFFFFFF45FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF43FFFFFFFFFFFF 1FFFFFFFFFF43FFFFFFFFFFFF 1FFFFFFFFFF42FFFFFFFFFFFF 1FFFFFFFFFF42FFFFFFFFFFFF 1FFFFFFFFFF41FFFFFFFFFFFF 1FFFFFFFFFF41FFFFFFFFFFFF 1FFFFFFFFFF40FFFFFFFFFFFF 1FFFFFFFFFF40FFFFFFFFFFFF 1FFFFFFFFFF3FFFFFFFFFFFFF 1FFFFFFFFFF3FFFFFFFFFFFFF 1FFFFFFFFFF3EFFFFFFFFFFFF 1FFFFFFFFFF3EFFFFFFFFFFFF 1FFFFFFFFFF3DFFFFFFFFFFFF 1FFFFFFFFFF3DFFFFFFFFFFFF 1FFFFFFFFFF3CFFFFFFFFFFFF 1FFFFFFFFFF3CFFFFFFFFFFFF 1FFFFFFFFFF3BFFFFFFFFFFFF 1FFFFFFFFFF3BFFFFFFFFFFFF 1FFFFFFFFFF3AFFFFFFFFFFFF 1FFFFFFFFFF3AFFFFFFFFFFFF 1FFFFFFFFFF39FFFFFFFFFFFF 1FFFFFFFFFF39FFFFFFFFFFFF 1FFFFFFFFFF38FFFFFFFFFFFF 1FFFFFFFFFF38FFFFFFFFFFFF 1FFFFFFFFFF37FFFFFFFFFFFF 1FFFFFFFFFF37FFFFFFFFFFFF 1FFFFFFFFFF36FFFFFFFFFFFF 1FFFFFFFFFF36FFFFFFFFFFFF 1FFFFFFFFFF35FFFFFFFFFFFF 1FFFFFFFFFF35FFFFFFFFFFFF 1FFFFFFFFFF34FFFFFFFFFFFF 1FFFFFFFFFF34FFFFFFFFFFFF 1FFFFFFFFFF33FFFFFFFFFFFF 1FFFFFFFFFF33FFFFFFFFFFFF 1FFFFFFFFFF32FFFFFFFFFFFF 1FFFFFFFFFF32FFFFFFFFFFFF 1FFFFFFFFFF31FFFFFFFFFFFF 1FFFFFFFFFF31FFFFFFFFFFFF 1FFFFFFFFFF30FFFFFFFFFFFF 1FFFFFFFFFF30FFFFFFFFFFFF 1FFFFFFFFFF2FFFFFFFFFFFFF 1FFFFFFFFFF2FFFFFFFFFFFFF 1FFFFFFFFFF2EFFFFFFFFFFFF 1FFFFFFFFFF2EFFFFFFFFFFFF 1FFFFFFFFFF2DFFFFFFFFFFFF 1FFFFFFFFFF2DFFFFFFFFFFFF 1FFFFFFFFFF2CFFFFFFFFFFFF 1FFFFFFFFFF2CFFFFFFFFFFFF 1FFFFFFFFFF2BFFFFFFFFFFFF 1FFFFFFFFFF2BFFFFFFFFFFFF 1FFFFFFFFFF2AFFFFFFFFFFFF 1FFFFFFFFFF2AFFFFFFFFFFFF 1FFFFFFFFFF29FFFFFFFFFFFF 1FFFFFFFFFF29FFFFFFFFFFFF 1FFFFFFFFFF28FFFFFFFFFFFF 1FFFFFFFFFF28FFFFFFFFFFFF 1FFFFFFFFFF27FFFFFFFFFFFF 1FFFFFFFFFF27FFFFFFFFFFFF 1FFFFFFFFFF26FFFFFFFFFFFF 1FFFFFFFFFF26FFFFFFFFFFFF 1FFFFFFFFFF25FFFFFFFFFFFF 1FFFFFFFFFF24FFFFFFFFFFFF 1FFFFFFFFFF24FFFFFFFFFFFF 1FFFFFFFFFF23FFFFFFFFFFFF 1FFFFFFFFFF23FFFFFFFFFFFF 1FFFFFFFFFF22FFFFFFFFFFFF 1FFFFFFFFFF22FFFFFFFFFFFF 1FFFFFFFFFF21FFFFFFFFFFFF 1FFFFFFFFFF21FFFFFFFFFFFF 1FFFFFFFFFF20FFFFFFFFFFFF 1FFFFFFFFFF20FFFFFFFFFFFF 1FFFFFFFFFF1FFFFFFFFFFFFF 1FFFFFFFFFF1FFFFFFFFFFFFF 1FFFFFFFFFF1EFFFFFFFFFFFF 1FFFFFFFFFF1EFFFFFFFFFFFF 1FFFFFFFFFF1DFFFFFFFFFFFF 1FFFFFFFFFF1DFFFFFFFFFFFF 1FFFFFFFFFF1CFFFFFFFFFFFF 1FFFFFFFFFF1CFFFFFFFFFFFF 1FFFFFFFFFF1BFFFFFFFFFFFF 1FFFFFFFFFF1BFFFFFFFFFFFF 1FFFFFFFFFF1AFFFFFFFFFFFF 1FFFFFFFFFF1AFFFFFFFFFFFF 1FFFFFFFFFF19FFFFFFFFFFFF 1FFFFFFFFFF18FFFFFFFFFFFF 1FFFFFFFFFF18FFFFFFFFFFFF 1FFFFFFFFFF17FFFFFFFFFFFF 1FFFFFFFFFF17FFFFFFFFFFFF 1FFFFFFFFFF16FFFFFFFFFFFF 1FFFFFFFFFF16FFFFFFFFFFFF 1FFFFFFFFFF15FFFFFFFFFFFF 1FFFFFFFFFF15FFFFFFFFFFFF 1FFFFFFFFFF14FFFFFFFFFFFF 1FFFFFFFFFF14FFFFFFFFFFFF 1FFFFFFFFFF13FFFFFFFFFFFF 1FFFFFFFFFF13FFFFFFFFFFFF 1FFFFFFFFFF12FFFFFFFFFFFF 1FFFFFFFFFF12FFFFFFFFFFFF 1FFFFFFFFFF11FFFFFFFFFFFF 1FFFFFFFFFF11FFFFFFFFFFFF 1FFFFFFFFFF10FFFFFFFFFFFF 1FFFFFFFFFF FFFFFFFFFFFFF 1FFFFFFFFFF FFFFFFFFFFFFF 1FFFFFFFFFF EFFFFFFFFFFFF 1FFFFFFFFFF EFFFFFFFFFFFF 1FFFFFFFFFF DFFFFFFFFFFFF 1FFFFFFFFFF DFFFFFFFFFFFF 1FFFFFFFFFF CFFFFFFFFFFFF 1FFFFFFFFFF CFFFFFFFFFFFF 1FFFFFFFFFF BFFFFFFFFFFFF 1FFFFFFFFFF BFFFFFFFFFFFF 1FFFFFFFFFF AFFFFFFFFFFFF 1FFFFFFFFFF 9FFFFFFFFFFFF 1FFFFFFFFFF 8FFFFFFFFFF12FFFFFFFF60FFFFFFFFFFFF 1FFFFFFFFFF 8FFFFFFFFFFFF 1FFFFFFFFFF 8FFFFFFFFFFFF 1FFFFFFFFFF 7FFFFFFFFFFFF 1FFFFFFFFFF 6FFFFFFFFFFFF 1FFFFFFFFFF 6FFFFFFFFFFFF 1FFFFFFFFFF 5FFFFFFFFFFFF 1FFFFFFFFFF 4FFFFFFFFFFFF 1FFFFFFFFFF 4FFFFFFFFFFFF 1FFFFFFFFFF 3FFFFFFFFFFFF 1FFFFFFFFFF 2FFFFFFFFFFFF 1FFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFF 7FFFFFFFFFFFF 1FFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFF 2FFFFFFFFFFFF 1FFFFFFFFFF 2FFFFFFFFFFFF 1FFFFFFFFFF 3FFFFFFFFFFFF 1FFFFFFFFFF 4FFFFFFFFFFFF 1FFFFFFFFFF 4FFFFFFFFFFFF 1FFFFFFFFFF 5FFFFFFFFFFFF 1FFFFFFFFFF 6FFFFFFFFFFFF 1FFFFFFFFFF 6FFFFFFFFFFFF 1FFFFFFFFFF 7FFFFFFFFFFFF 1FFFFFFFFFF 8FFFFFFFFFFFF 1FFFFFFFFFF 8FFFFFFFFFFFF 1FFFFFFFFFF 9FFFFFFFFFFFF 1FFFFFFFFFF AFFFFFFFFFFFF 1FFFFFFFFFF AFFFFFFFFFFFF 1FFFFFFFFFF BFFFFFFFFFFFF 1FFFFFFFFFF CFFFFFFFFFFFF 1FFFFFFFFFF CFFFFFFFFFFFF 1FFFFFFFFFF DFFFFFFFFFFFF 1FFFFFFFFFF EFFFFFFFFFFFF 1FFFFFFFFFF EFFFFFFFFFFFF 1FFFFFFFFFF FFFFFFFFFFFFF 1FFFFFFFFFF FFFFFFFFFFFFF 1FFFFFFFFFF10FFFFFFFFFFFF 1FFFFFFFFFF11FFFFFFFFFFFF 1FFFFFFFFFF11FFFFFFFFFFFF 1FFFFFFFFFF12FFFFFFFFFFFF 1FFFFFFFFFF13FFFFFFFFFFFF 1FFFFFFFFFF13FFFFFFFFFFFF 1FFFFFFFFFF14FFFFFFFFFFFF 1FFFFFFFFFF15FFFFFFFFFFFF 1FFFFFFFFFF15FFFFFFFFFFFF 1FFFFFFFFFF16FFFFFFFFFFFF 1FFFFFFFFFF17FFFFFFFFFFFF 1FFFFFFFFFF17FFFFFFFFFFFF 1FFFFFFFFFF18FFFFFFFFFFFF 1FFFFFFFFFF18FFFFFFFFFFFF 1FFFFFFFFFF19FFFFFFFFFFFF 1FFFFFFFFFF1AFFFFFFFFFFFF 1FFFFFFFFFF1AFFFFFFFFFFFF 1FFFFFFFFFF1BFFFFFFFFFFFF 1FFFFFFFFFF1CFFFFFFFFFFFF 1FFFFFFFFFF1CFFFFFFFFFFFF 1FFFFFFFFFF1DFFFFFFFFFFFF 1FFFFFFFFFF1EFFFFFFFFFFFF 1FFFFFFFFFF1EFFFFFFFFFFFF 1FFFFFFFFFF1FFFFFFFFFFFFF 1FFFFFFFFFF1FFFFFFFFFFFFF 1FFFFFFFFFF20FFFFFFFFFFFF 1FFFFFFFFFF21FFFFFFFFFFFF 1FFFFFFFFFF21FFFFFFFFFFFF 1FFFFFFFFFF22FFFFFFFFFFFF 1FFFFFFFFFF23FFFFFFFFFFFF 1FFFFFFFFFF23FFFFFFFFFFFF 1FFFFFFFFFF24FFFFFFFFFFFF 1FFFFFFFFFF24FFFFFFFFFFFF 1FFFFFFFFFF25FFFFFFFFFFFF 1FFFFFFFFFF26FFFFFFFFFFFF 1FFFFFFFFFF26FFFFFFFFFFFF 1FFFFFFFFFF27FFFFFFFFFFFF 1FFFFFFFFFF27FFFFFFFFFFFF 1FFFFFFFFFF28FFFFFFFFFFFF 1FFFFFFFFFF29FFFFFFFFFFFF 1FFFFFFFFFF29FFFFFFFFFFFF 1FFFFFFFFFF2AFFFFFFFFFFFF 1FFFFFFFFFF2AFFFFFFFFFFFF 1FFFFFFFFFF2BFFFFFFFFFFFF 1FFFFFFFFFF2CFFFFFFFFFFFF 1FFFFFFFFFF2CFFFFFFFFFFFF 1FFFFFFFFFF2DFFFFFFFFFFFF 1FFFFFFFFFF2EFFFFFFFFFFFF 1FFFFFFFFFF2EFFFFFFFFFFFF 1FFFFFFFFFF2FFFFFFFFFFFFF 1FFFFFFFFFF2FFFFFFFFFFFFF 1FFFFFFFFFF30FFFFFFFFFFFF 1FFFFFFFFFF30FFFFFFFFFFFF 1FFFFFFFFFF31FFFFFFFFFFFF 1FFFFFFFFFF32FFFFFFFFFFFF 1FFFFFFFFFF32FFFFFFFFFFFF 1FFFFFFFFFF33FFFFFFFFFFFF 1FFFFFFFFFF33FFFFFFFFFFFF 1FFFFFFFFFF34FFFFFFFFFFFF 1FFFFFFFFFF35FFFFFFFFFFFF 1FFFFFFFFFF35FFFFFFFFFFFF 1FFFFFFFFFF36FFFFFFFFFFFF 1FFFFFFFFFF36FFFFFFFFFFFF 1FFFFFFFFFF37FFFFFFFFFFFF 1FFFFFFFFFF38FFFFFFFFFFFF 1FFFFFFFFFF38FFFFFFFFFFFF 1FFFFFFFFFF39FFFFFFFFFFFF 1FFFFFFFFFF39FFFFFFFFFFFF 1FFFFFFFFFF3AFFFFFFFFFFFF 1FFFFFFFFFF3AFFFFFFFFFFFF 1FFFFFFFFFF3BFFFFFFFFFFFF 1FFFFFFFFFF3CFFFFFFFFFFFF 1FFFFFFFFFF3CFFFFFFFFFFFF 1FFFFFFFFFF3DFFFFFFFFFFFF 1FFFFFFFFFF3DFFFFFFFFFFFF 1FFFFFFFFFF3EFFFFFFFFFFFF 1FFFFFFFFFF3EFFFFFFFFFFFF 1FFFFFFFFFF3FFFFFFFFFFFFF 1FFFFFFFFFF40FFFFFFFFFFFF 1FFFFFFFFFF40FFFFFFFFFFFF 1FFFFFFFFFF41FFFFFFFFFFFF 1FFFFFFFFFF41FFFFFFFFFFFF 1FFFFFFFFFF42FFFFFFFFFFFF 1FFFFFFFFFF42FFFFFFFFFFFF 1FFFFFFFFFF43FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF45FFFFFFFFFFFF 1FFFFFFFFFF45FFFFFFFFFFFF 1FFFFFFFFFF46FFFFFFFFFFFF 1FFFFFFFFFF46FFFFFFFFFFFF 1FFFFFFFFFF47FFFFFFFFFFFF 1FFFFFFFFFF47FFFFFFFFFFFF 1FFFFFFFFFF48FFFFFFFFFFFF 1FFFFFFFFFF49FFFFFFFFFFFF 1FFFFFFFFFF49FFFFFFFFFFFF 1FFFFFFFFFF4AFFFFFFFFFFFF 1FFFFFFFFFF4AFFFFFFFFFFFF 1FFFFFFFFFF4BFFFFFFFFFFFF 1FFFFFFFFFF4BFFFFFFFFFFFF 1FFFFFFFFFF4CFFFFFFFFFFFF 1FFFFFFFFFF4CFFFFFFFFFFFF 1FFFFFFFFFF4DFFFFFFFFFFFF 1FFFFFFFFFF4DFFFFFFFFFFFF 1FFFFFFFFFF4EFFFFFFFFFFFF 1FFFFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF50FFFFFFFFFFFF 1FFFFFFFFFF50FFFFFFFFFFFF 1FFFFFFFFFF51FFFFFFFFFFFF 1FFFFFFFFFF51FFFFFFFFFFFF 1FFFFFFFFFF52FFFFFFFFFFFF 1FFFFFFFFFF52FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF54FFFFFFFFFFFF 1FFFFFFFFFF54FFFFFFFFFFFF 1FFFFFFFFFF55FFFFFFFFFFFF 1FFFFFFFFFF55FFFFFFFFFFFF 1FFFFFFFFFF56FFFFFFFFFFFF 1FFFFFFFFFF57FFFFFFFFFFFF 1FFFFFFFFFF57FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF59FFFFFFFFFFFF 1FFFFFFFFFF59FFFFFFFFFFFF 1FFFFFFFFFF5AFFFFFFFFFFFF 1FFFFFFFFFF5AFFFFFFFFFFFF 1FFFFFFFFFF5BFFFFFFFFFFFF 1FFFFFFFFFF5BFFFFFFFFFFFF 1FFFFFFFFFF5CFFFFFFFFFFFF 1FFFFFFFFFF5CFFFFFFFFFFFF 1FFFFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF5EFFFFFFFFFFFF 1FFFFFFFFFF5EFFFFFFFFFFFF 1FFFFFFFFFF5FFFFFFFFFFFFF 1FFFFFFFFFF5FFFFFFFFFFFFF 1FFFFFFFFFF60FFFFFFFFFFFF 1FFFFFFFFFF60FFFFFFFFFFFF 1FFFFFFFFFF61FFFFFFFFFFFF 1FFFFFFFFFF61FFFFFFFFFFFF 1FFFFFFFFFF62FFFFFFFFFFFF 1FFFFFFFFFF62FFFFFFFFFFFF 1FFFFFFFFFF63FFFFFFFFFFFF 1FFFFFFFFFF63FFFFFFFFFFFF 1FFFFFFFFFF64FFFFFFFFFFFF 1FFFFFFFFFF64FFFFFFFFFFFF 1FFFFFFFFFF65FFFFFFFFFFFF 1FFFFFFFFFF65FFFFFFFFFFFF 1FFFFFFFFFF66FFFFFFFFFFFF 1FFFFFFFFFF66FFFFFFFFFFFF 1FFFFFFFFFF67FFFFFFFFFFFF 1FFFFFFFFFF67FFFFFFFFFFFF 1FFFFFFFFFF68FFFFFFFFFFFF 1FFFFFFFFFF68FFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF6AFFFFFFFFFFFF 1FFFFFFFFFF6AFFFFFFFFFFFF 1FFFFFFFFFF6BFFFFFFFFFFFF 1FFFFFFFFFF6BFFFFFFFFFFFF 1FFFFFFFFFF6CFFFFFFFFFFFF 1FFFFFFFFFF6CFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF70FFFFFFFFFFFF 1FFFFFFFFFF70FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF72FFFFFFFFFFFF 1FFFFFFFFFF72FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF77FFFFFFFFFFFF 1FFFFFFFFFF77FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF79FFFFFFFFFFFF 1FFFFFFFFFF79FFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7DFFFFFFFFFFFF 1FFFFFFFFFF7DFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7DFFFFFFFFFFFF 1FFFFFFFFFF7DFFFFFFFFFFFF 1FFFFFFFFFF7DFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF79FFFFFFFFFFFF 1FFFFFFFFFF79FFFFFFFFFFFF 1FFFFFFFFFF79FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF77FFFFFFFFFFFF 1FFFFFFFFFF77FFFFFFFFFFFF 1FFFFFFFFFF77FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF72FFFFFFFFFFFF 1FFFFFFFFFF72FFFFFFFFFFFF 1FFFFFFFFFF72FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF70FFFFFFFFFFFF 1FFFFFFFFFF70FFFFFFFFFFFF 1FFFFFFFFFF70FFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6CFFFFFFFFFFFF 1FFFFFFFFFF6CFFFFFFFFFFFF 1FFFFFFFFFF6CFFFFFFFFFFFF 1FFFFFFFFFF6BFFFFFFFFFFFF 1FFFFFFFFFF6BFFFFFFFFFFFF 1FFFFFFFFFF6BFFFFFFFFFFFF 1FFFFFFFFFF6AFFFFFFFFFFFF 1FFFFFFFFFF6AFFFFFFFFFFFF 1FFFFFFFFFF6AFFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF68FFFFFFFFFFFF 1FFFFFFFFFF68FFFFFFFFFFFF 1FFFFFFFFFF68FFFFFFFFFFFF 1FFFFFFFFFF67FFFFFFFFFFFF 1FFFFFFFFFF67FFFFFFFFFFFF 1FFFFFFFFFF67FFFFFFFFFFFF 1FFFFFFFFFF66FFFFFFFFFFFF 1FFFFFFFFFF66FFFFFFFFFFFF 1FFFFFFFFFF66FFFFFFFFFFFF 1FFFFFFFFFF65FFFFFFFFFFFF 1FFFFFFFFFF65FFFFFFFFFFFF 1FFFFFFFFFF65FFFFFFFFFFFF 1FFFFFFFFFF64FFFFFFFFFFFF 1FFFFFFFFFF64FFFFFFFFFFFF 1FFFFFFFFFF64FFFFFFFFFFFF 1FFFFFFFFFF63FFFFFFFFFFFF 1FFFFFFFFFF63FFFFFFFFFFFF 1FFFFFFFFFF63FFFFFFFFFFFF 1FFFFFFFFFF62FFFFFFFFFFFF 1FFFFFFFFFF62FFFFFFFFFFFF 1FFFFFFFFFF62FFFFFFFFFFFF 1FFFFFFFFFF61FFFFFFFFFFFF 1FFFFFFFFFF61FFFFFFFFFFFF 1FFFFFFFFFF61FFFFFFFFFFFF 1FFFFFFFFFF60FFFFFFFFFFFF 1FFFFFFFFFF60FFFFFFFFFFFF 1FFFFFFFFFF60FFFFFFFFFFFF 1FFFFFFFFFF5FFFFFFFFFFFFF 1FFFFFFFFFF5FFFFFFFFFFFFF 1FFFFFFFFFF5FFFFFFFFFFFFF 1FFFFFFFFFF5EFFFFFFFFFFFF 1FFFFFFFFFF5EFFFFFFFFFFFF 1FFFFFFFFFF5EFFFFFFFFFFFF 1FFFFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF5CFFFFFFFFFFFF 1FFFFFFFFFF5CFFFFFFFFFFFF 1FFFFFFFFFF5BFFFFFFFFFFFF 1FFFFFFFFFF5BFFFFFFFFFFFF 1FFFFFFFFFF5BFFFFFFFFFFFF 1FFFFFFFFFF5AFFFFFFFFFFFF 1FFFFFFFFFF5AFFFFFFFFFFFF 1FFFFFFFFFF5AFFFFFFFFFFFF 1FFFFFFFFFF59FFFFFFFFFFFF 1FFFFFFFFFF59FFFFFFFFFFFF 1FFFFFFFFFF59FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF57FFFFFFFFFFFF 1FFFFFFFFFF57FFFFFFFFFFFF 1FFFFFFFFFF57FFFFFFFFFFFF 1FFFFFFFFFF56FFFFFFFFFFFF 1FFFFFFFFFF56FFFFFFFFFFFF 1FFFFFFFFFF56FFFFFFFFFFFF 1FFFFFFFFFF55FFFFFFFFFFFF 1FFFFFFFFFF55FFFFFFFFFFFF 1FFFFFFFFFF55FFFFFFFFFFFF 1FFFFFFFFFF54FFFFFFFFFFFF 1FFFFFFFFFF54FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF52FFFFFFFFFFFF 1FFFFFFFFFF52FFFFFFFFFFFF 1FFFFFFFFFF52FFFFFFFFFFFF 1FFFFFFFFFF51FFFFFFFFFFFF 1FFFFFFFFFF51FFFFFFFFFFFF 1FFFFFFFFFF51FFFFFFFFFFFF 1FFFFFFFFFF50FFFFFFFFFFFF 1FFFFFFFFFF50FFFFFFFFFFFF 1FFFFFFFFFF50FFFFFFFFFFFF 1FFFFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF4EFFFFFFFFFFFF 1FFFFFFFFFF4EFFFFFFFFFFFF 1FFFFFFFFFF4DFFFFFFFFFFFF 1FFFFFFFFFF4DFFFFFFFFFFFF 1FFFFFFFFFF4DFFFFFFFFFFFF 1FFFFFFFFFF4CFFFFFFFFFFFF 1FFFFFFFFFF4CFFFFFFFFFFFF 1FFFFFFFFFF4CFFFFFFFFFFFF 1FFFFFFFFFF4BFFFFFFFFFFFF 1FFFFFFFFFF4BFFFFFFFFFFFF 1FFFFFFFFFF4BFFFFFFFFFFFF 1FFFFFFFFFF4AFFFFFFFFFFFF 1FFFFFFFFFF4AFFFFFFFFFFFF 1FFFFFFFFFF4AFFFFFFFFFFFF 1FFFFFFFFFF49FFFFFFFFFFFF 1FFFFFFFFFF49FFFFFFFFFFFF 1FFFFFFFFFF48FFFFFFFFFFFF 1FFFFFFFFFF48FFFFFFFFFFFF 1FFFFFFFFFF48FFFFFFFFFFFF 1FFFFFFFFFF47FFFFFFFFFFFF 1FFFFFFFFFF47FFFFFFFFFFFF 1FFFFFFFFFF47FFFFFFFFFFFF 1FFFFFFFFFF46FFFFFFFFFFFF 1FFFFFFFFFF46FFFFFFFFFFFF 1FFFFFFFFFF46FFFFFFFFFFFF 1FFFFFFFFFF45FFFFFFFFFFFF 1FFFFFFFFFF45FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF43FFFFFFFFFFFF 1FFFFFFFFFF43FFFFFFFFFFFF 1FFFFFFFFFF43FFFFFFFFFFFF 1FFFFFFFFFF42FFFFFFFFFFFF 1FFFFFFFFFF42FFFFFFFFFFFF 1FFFFFFFFFF42FFFFFFFFFFFF 1FFFFFFFFFF41FFFFFFFFFFFF 1FFFFFFFFFF41FFFFFFFFFFFF 1FFFFFFFFFF40FFFFFFFFFFFF 1FFFFFFFFFF40FFFFFFFFFFFF 1FFFFFFFFFF40FFFFFFFFFFFF 1FFFFFFFFFF3FFFFFFFFFFFFF 1FFFFFFFFFF3FFFFFFFFFFFFF 1FFFFFFFFFF3FFFFFFFFFFFFF 1FFFFFFFFFF3EFFFFFFFFFFFF 1FFFFFFFFFF3EFFFFFFFFFFFF 1FFFFFFFFFF3EFFFFFFFFFFFF 1FFFFFFFFFF3DFFFFFFFFFFFF 1FFFFFFFFFF3DFFFFFFFFFFFF 1FFFFFFFFFF3CFFFFFFFFFFFF 1FFFFFFFFFF3CFFFFFFFFFFFF 1FFFFFFFFFF3CFFFFFFFFFFFF 1FFFFFFFFFF3BFFFFFFFFFFFF 1FFFFFFFFFF3BFFFFFFFFFFFF 1FFFFFFFFFF3BFFFFFFFFFFFF 1FFFFFFFFFF3AFFFFFFFFFFFF 1FFFFFFFFFF3AFFFFFFFFFFFF 1FFFFFFFFFF39FFFFFFFFFFFF 1FFFFFFFFFF39FFFFFFFFFFFF 1FFFFFFFFFF39FFFFFFFFFFFF 1FFFFFFFFFF38FFFFFFFFFFFF 1FFFFFFFFFF38FFFFFFFFFFFF 1FFFFFFFFFF38FFFFFFFFFFFF 1FFFFFFFFFF37FFFFFFFFFFFF 1FFFFFFFFFF37FFFFFFFFFFFF 1FFFFFFFFFF36FFFFFFFFFFFF 1FFFFFFFFFF36FFFFFFFFFFFF 1FFFFFFFFFF36FFFFFFFFFFFF 1FFFFFFFFFF35FFFFFFFFFFFF 1FFFFFFFFFF35FFFFFFFFFFFF 1FFFFFFFFFF35FFFFFFFFFFFF 1FFFFFFFFFF34FFFFFFFFFFFF 1FFFFFFFFFF34FFFFFFFFFFFF 1FFFFFFFFFF33FFFFFFFFFFFF 1FFFFFFFFFF33FFFFFFFFFFFF 1FFFFFFFFFF33FFFFFFFFFFFF 1FFFFFFFFFF32FFFFFFFFFFFF 1FFFFFFFFFF32FFFFFFFFFFFF 1FFFFFFFFFF32FFFFFFFFFFFF 1FFFFFFFFFF31FFFFFFFFFFFF 1FFFFFFFFFF31FFFFFFFFFFFF 1FFFFFFFFFF30FFFFFFFFFFFF 1FFFFFFFFFF30FFFFFFFFFFFF 1FFFFFFFFFF30FFFFFFFFFFFF 1FFFFFFFFFF2FFFFFFFFFFFFF 1FFFFFFFFFF2FFFFFFFFFFFFF 1FFFFFFFFFF2FFFFFFFFFFFFF 1FFFFFFFFFF2EFFFFFFFFFFFF 1FFFFFFFFFF2EFFFFFFFFFFFF 1FFFFFFFFFF2DFFFFFFFFFFFF 1FFFFFFFFFF2DFFFFFFFFFFFF 1FFFFFFFFFF2DFFFFFFFFFFFF 1FFFFFFFFFF2CFFFFFFFFFFFF 1FFFFFFFFFF2CFFFFFFFFFFFF 1FFFFFFFFFF2BFFFFFFFFFFFF 1FFFFFFFFFF2BFFFFFFFFFFFF 1FFFFFFFFFF2BFFFFFFFFFFFF 1FFFFFFFFFF2AFFFFFFFFFFFF 1FFFFFFFFFF2AFFFFFFFFFFFF 1FFFFFFFFFF2AFFFFFFFFFFFF 1FFFFFFFFFF29FFFFFFFFFFFF 1FFFFFFFFFF29FFFFFFFFFFFF 1FFFFFFFFFF28FFFFFFFFFFFF 1FFFFFFFFFF28FFFFFFFFFFFF 1FFFFFFFFFF28FFFFFFFFFFFF 1FFFFFFFFFF27FFFFFFFFFFFF 1FFFFFFFFFF27FFFFFFFFFFFF 1FFFFFFFFFF26FFFFFFFFFFFF 1FFFFFFFFFF26FFFFFFFFFFFF 1FFFFFFFFFF26FFFFFFFFFFFF 1FFFFFFFFFF25FFFFFFFFFFFF 1FFFFFFFFFF25FFFFFFFFFFFF 1FFFFFFFFFF24FFFFFFFFFFFF 1FFFFFFFFFF24FFFFFFFFFFFF 1FFFFFFFFFF24FFFFFFFFFFFF 1FFFFFFFFFF23FFFFFFFFFFFF 1FFFFFFFFFF23FFFFFFFFFFFF 1FFFFFFFFFF23FFFFFFFFFFFF 1FFFFFFFFFF22FFFFFFFFFFFF 1FFFFFFFFFF22FFFFFFFFFFFF 1FFFFFFFFFF21FFFFFFFFFFFF 1FFFFFFFFFF21FFFFFFFFFFFF 1FFFFFFFFFF21FFFFFFFFFFFF 1FFFFFFFFFF20FFFFFFFFFFFF 1FFFFFFFFFF20FFFFFFFFFFFF 1FFFFFFFFFF1FFFFFFFFFFFFF 1FFFFFFFFFF1FFFFFFFFFFFFF 1FFFFFFFFFF1FFFFFFFFFFFFF 1FFFFFFFFFF1EFFFFFFFFFFFF 1FFFFFFFFFF1EFFFFFFFFFFFF 1FFFFFFFFFF1DFFFFFFFFFFFF 1FFFFFFFFFF1DFFFFFFFFFFFF 1FFFFFFFFFF1DFFFFFFFFFFFF 1FFFFFFFFFF1CFFFFFFFFFFFF 1FFFFFFFFFF1CFFFFFFFFFFFF 1FFFFFFFFFF1BFFFFFFFFFFFF 1FFFFFFFFFF1BFFFFFFFFFFFF 1FFFFFFFFFF1BFFFFFFFFFFFF 1FFFFFFFFFF1AFFFFFFFFFFFF 1FFFFFFFFFF1AFFFFFFFFFFFF 1FFFFFFFFFF19FFFFFFFFFFFF 1FFFFFFFFFF19FFFFFFFFFFFF 1FFFFFFFFFF19FFFFFFFFFFFF 1FFFFFFFFFF18FFFFFFFFFFFF 1FFFFFFFFFF18FFFFFFFFFFFF 1FFFFFFFFFF18FFFFFFFFFFFF 1FFFFFFFFFF17FFFFFFFFFFFF 1FFFFFFFFFF17FFFFFFFFFFFF 1FFFFFFFFFF16FFFFFFFFFFFF 1FFFFFFFFFF16FFFFFFFFFFFF 1FFFFFFFFFF16FFFFFFFFFFFF 1FFFFFFFFFF15FFFFFFFFFFFF 1FFFFFFFFFF15FFFFFFFFFFFF 1FFFFFFFFFF14FFFFFFFFFFFF 1FFFFFFFFFF14FFFFFFFFFFFF 1FFFFFFFFFF14FFFFFFFFFFFF 1FFFFFFFFFF13FFFFFFFFFFFF 1FFFFFFFFFF13FFFFFFFFFFFF 1FFFFFFFFFF12FFFFFFFFFFFF 1FFFFFFFFFF12FFFFFFFFFFFF 1FFFFFFFFFF11FFFFFFFFFFFF 1FFFFFFFFFF11FFFFFFFFFFFF 1FFFFFFFFFF11FFFFFFFFFFFF 1FFFFFFFFFF10FFFFFFFFFFFF 1FFFFFFFFFF10FFFFFFFFFFFF 1FFFFFFFFFF FFFFFFFFFFFFF 1FFFFFFFFFF FFFFFFFFFFFFF 1FFFFFFFFFF FFFFFFFFFFFFF 1FFFFFFFFFF EFFFFFFFFFFFF 1FFFFFFFFFF EFFFFFFFFFFFF 1FFFFFFFFFF DFFFFFFFFFFFF 1FFFFFFFFFF DFFFFFFFFFFFF 1FFFFFFFFFF DFFFFFFFFFFFF 1FFFFFFFFFF CFFFFFFFFFFFF 1FFFFFFFFFF CFFFFFFFFFFFF 1FFFFFFFFFF BFFFFFFFFFFFF 1FFFFFFFFFF BFFFFFFFFFFFF 1FFFFFFFFFF BFFFFFFFFFFFF 1FFFFFFFFFF AFFFFFFFFFFFF 1FFFFFFFFFF AFFFFFFFFFFFF 1FFFFFFFFFF 9FFFFFFFFFFFF 1FFFFFFFFFF 9FFFFFFFFFFFF 1FFFFFFFFFF 9FFFFFFFFFFFF 1FFFFFFFFFF 8FFFFFFFFFFFF 1FFFFFFFFFF 8FFFFFFFFFFFF 1FFFFFFFFFF 7FFFFFFFFFFFF 1FFFFFFFFFF 7FFFFFFFFFFFF 1FFFFFFFFFF 7FFFFFFFFFFFF 1FFFFFFFFFF 6FFFFFFFFFFFF 1FFFFFFFFFF 6FFFFFFFFFFFF 1FFFFFFFFFF 5FFFFFFFFFFFF 1FFFFFFFFFF 5FFFFFFFFFFFF 1FFFFFFFFFF 4FFFFFFFFFFFF 1FFFFFFFFFF 4FFFFFFFFFFFF 1FFFFFFFFFF 4FFFFFFFFFFFF 1FFFFFFFFFF 3FFFFFFFFFFFF 1FFFFFFFFFF 3FFFFFFFFFFFF 1FFFFFFFFFF 2FFFFFFFFFFFF 1FFFFFFFFFF 2FFFFFFFFFFFF 1FFFFFFFFFF 2FFFFFFFFFFFF 1FFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFF67FFFFFFFFFFFF 1FFFFFFFF32FFFFFFFFFFFF 1FFFFFFFF 3FFFFFFFFFFFF 1FFFFFFFF38FFFFFFFFFFFF 1FFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFF 2FFFFFFFFFFFF 1FFFFFFFFFF 2FFFFFFFFFFFF 1FFFFFFFFFF 2FFFFFFFFFFFF 1FFFFFFFFFF 3FFFFFFFFFFFF 1FFFFFFFFFF 3FFFFFFFFFFFF 1FFFFFFFFFF 4FFFFFFFFFFFF 1FFFFFFFFFF 4FFFFFFFFFFFF 1FFFFFFFFFF 5FFFFFFFFFFFF 1FFFFFFFFFF 5FFFFFFFFFFFF 1FFFFFFFFFF 5FFFFFFFFFFFF 1FFFFFFFFFF 6FFFFFFFFFFFF 1FFFFFFFFFF 6FFFFFFFFFFFF 1FFFFFFFFFF 7FFFFFFFFFFFF 1FFFFFFFFFF 7FFFFFFFFFFFF 1FFFFFFFFFF 7FFFFFFFFFFFF 1FFFFFFFFFF 8FFFFFFFFFFFF 1FFFFFFFFFF 8FFFFFFFFFFFF 1FFFFFFFFFF 9FFFFFFFFFFFF 1FFFFFFFFFF 9FFFFFFFFFFFF 1FFFFFFFFFF AFFFFFFFFFFFF 1FFFFFFFFFF AFFFFFFFFFFFF 1FFFFFFFFFF AFFFFFFFFFFFF 1FFFFFFFFFF BFFFFFFFFFFFF 1FFFFFFFFFF BFFFFFFFFFFFF 1FFFFFFFFFF CFFFFFFFFFFFF 1FFFFFFFFFF CFFFFFFFFFFFF 1FFFFFFFFFF DFFFFFFFFFFFF 1FFFFFFFFFF DFFFFFFFFFFFF 1FFFFFFFFFF DFFFFFFFFFFFF 1FFFFFFFFFF EFFFFFFFFFFFF 1FFFFFFFFFF EFFFFFFFFFFFF 1FFFFFFFFFF FFFFFFFFFFFFF 1FFFFFFFFFF FFFFFFFFFFFFF 1FFFFFFFFFF10FFFFFFFFFFFF 1FFFFFFFFFF10FFFFFFFFFFFF 1FFFFFFFFFF10FFFFFFFFFFFF 1FFFFFFFFFF11FFFFFFFFFFFF 1FFFFFFFFFF11FFFFFFFFFFFF 1FFFFFFFFFF12FFFFFFFFFFFF 1FFFFFFFFFF12FFFFFFFFFFFF 1FFFFFFFFFF13FFFFFFFFFFFF 1FFFFFFFFFF13FFFFFFFFFFFF 1FFFFFFFFFF13FFFFFFFFFFFF 1FFFFFFFFFF14FFFFFFFFFFFF 1FFFFFFFFFF14FFFFFFFFFFFF 1FFFFFFFFFF15FFFFFFFFFFFF 1FFFFFFFFFF15FFFFFFFFFFFF 1FFFFFFFFFF16FFFFFFFFFFFF 1FFFFFFFFFF16FFFFFFFFFFFF 1FFFFFFFFFF16FFFFFFFFFFFF 1FFFFFFFFFF17FFFFFFFFFFFF 1FFFFFFFFFF17FFFFFFFFFFFF 1FFFFFFFFFF18FFFFFFFFFFFF 1FFFFFFFFFF18FFFFFFFFFFFF 1FFFFFFFFFF19FFFFFFFFFFFF 1FFFFFFFFFF19FFFFFFFFFFFF 1FFFFFFFFFF19FFFFFFFFFFFF 1FFFFFFFFFF1AFFFFFFFFFFFF 1FFFFFFFFFF1AFFFFFFFFFFFF 1FFFFFFFFFF1BFFFFFFFFFFFF 1FFFFFFFFFF1BFFFFFFFFFFFF 1FFFFFFFFFF1CFFFFFFFFFFFF 1FFFFFFFFFF1CFFFFFFFFFFFF 1FFFFFFFFFF1DFFFFFFFFFFFF 1FFFFFFFFFF1DFFFFFFFFFFFF 1FFFFFFFFFF1DFFFFFFFFFFFF 1FFFFFFFFFF1EFFFFFFFFFFFF 1FFFFFFFFFF1EFFFFFFFFFFFF 1FFFFFFFFFF1FFFFFFFFFFFFF 1FFFFFFFFFF1FFFFFFFFFFFFF 1FFFFFFFFFF20FFFFFFFFFFFF 1FFFFFFFFFF20FFFFFFFFFFFF 1FFFFFFFFFF20FFFFFFFFFFFF 1FFFFFFFFFF21FFFFFFFFFFFF 1FFFFFFFFFF21FFFFFFFFFFFF 1FFFFFFFFFF22FFFFFFFFFFFF 1FFFFFFFFFF22FFFFFFFFFFFF 1FFFFFFFFFF23FFFFFFFFFFFF 1FFFFFFFFFF23FFFFFFFFFFFF 1FFFFFFFFFF24FFFFFFFFFFFF 1FFFFFFFFFF24FFFFFFFFFFFF 1FFFFFFFFFF24FFFFFFFFFFFF 1FFFFFFFFFF25FFFFFFFFFFFF 1FFFFFFFFFF25FFFFFFFFFFFF 1FFFFFFFFFF26FFFFFFFFFFFF 1FFFFFFFFFF26FFFFFFFFFFFF 1FFFFFFFFFF27FFFFFFFFFFFF 1FFFFFFFFFF27FFFFFFFFFFFF 1FFFFFFFFFF27FFFFFFFFFFFF 1FFFFFFFFFF28FFFFFFFFFFFF 1FFFFFFFFFF28FFFFFFFFFFFF 1FFFFFFFFFF29FFFFFFFFFFFF 1FFFFFFFFFF29FFFFFFFFFFFF 1FFFFFFFFFF2AFFFFFFFFFFFF 1FFFFFFFFFF2AFFFFFFFFFFFF 1FFFFFFFFFF2BFFFFFFFFFFFF 1FFFFFFFFFF2BFFFFFFFFFFFF 1FFFFFFFFFF2CFFFFFFFFFFFF 1FFFFFFFFFF2CFFFFFFFFFFFF 1FFFFFFFFFF2CFFFFFFFFFFFF 1FFFFFFFFFF2DFFFFFFFFFFFF 1FFFFFFFFFF2DFFFFFFFFFFFF 1FFFFFFFFFF2EFFFFFFFFFFFF 1FFFFFFFFFF2EFFFFFFFFFFFF 1FFFFFFFFFF2FFFFFFFFFFFFF 1FFFFFFFFFF2FFFFFFFFFFFFF 1FFFFFFFFFF30FFFFFFFFFFFF 1FFFFFFFFFF30FFFFFFFFFFFF 1FFFFFFFFFF30FFFFFFFFFFFF 1FFFFFFFFFF31FFFFFFFFFFFF 1FFFFFFFFFF31FFFFFFFFFFFF 1FFFFFFFFFF32FFFFFFFFFFFF 1FFFFFFFFFF32FFFFFFFFFFFF 1FFFFFFFFFF33FFFFFFFFFFFF 1FFFFFFFFFF33FFFFFFFFFFFF 1FFFFFFFFFF34FFFFFFFFFFFF 1FFFFFFFFFF34FFFFFFFFFFFF 1FFFFFFFFFF34FFFFFFFFFFFF 1FFFFFFFFFF35FFFFFFFFFFFF 1FFFFFFFFFF35FFFFFFFFFFFF 1FFFFFFFFFF36FFFFFFFFFFFF 1FFFFFFFFFF36FFFFFFFFFFFF 1FFFFFFFFFF37FFFFFFFFFFFF 1FFFFFFFFFF37FFFFFFFFFFFF 1FFFFFFFFFF38FFFFFFFFFFFF 1FFFFFFFFFF38FFFFFFFFFFFF 1FFFFFFFFFF39FFFFFFFFFFFF 1FFFFFFFFFF39FFFFFFFFFFFF 1FFFFFFFFFF39FFFFFFFFFFFF 1FFFFFFFFFF3AFFFFFFFFFFFF 1FFFFFFFFFF3AFFFFFFFFFFFF 1FFFFFFFFFF3BFFFFFFFFFFFF 1FFFFFFFFFF3BFFFFFFFFFFFF 1FFFFFFFFFF3CFFFFFFFFFFFF 1FFFFFFFFFF3CFFFFFFFFFFFF 1FFFFFFFFFF3DFFFFFFFFFFFF 1FFFFFFFFFF3DFFFFFFFFFFFF 1FFFFFFFFFF3EFFFFFFFFFFFF 1FFFFFFFFFF3EFFFFFFFFFFFF 1FFFFFFFFFF3FFFFFFFFFFFFF 1FFFFFFFFFF3FFFFFFFFFFFFF 1FFFFFFFFFF3FFFFFFFFFFFFF 1FFFFFFFFFF40FFFFFFFFFFFF 1FFFFFFFFFF40FFFFFFFFFFFF 1FFFFFFFFFF41FFFFFFFFFFFF 1FFFFFFFFFF41FFFFFFFFFFFF 1FFFFFFFFFF42FFFFFFFFFFFF 1FFFFFFFFFF42FFFFFFFFFFFF 1FFFFFFFFFF43FFFFFFFFFFFF 1FFFFFFFFFF43FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF44FFFFFFFFFFFF 1FFFFFFFFFF45FFFFFFFFFFFF 1FFFFFFFFFF45FFFFFFFFFFFF 1FFFFFFFFFF46FFFFFFFFFFFF 1FFFFFFFFFF46FFFFFFFFFFFF 1FFFFFFFFFF47FFFFFFFFFFFF 1FFFFFFFFFF47FFFFFFFFFFFF 1FFFFFFFFFF48FFFFFFFFFFFF 1FFFFFFFFFF48FFFFFFFFFFFF 1FFFFFFFFFF49FFFFFFFFFFFF 1FFFFFFFFFF49FFFFFFFFFFFF 1FFFFFFFFFF4AFFFFFFFFFFFF 1FFFFFFFFFF4AFFFFFFFFFFFF 1FFFFFFFFFF4BFFFFFFFFFFFF 1FFFFFFFFFF4BFFFFFFFFFFFF 1FFFFFFFFFF4BFFFFFFFFFFFF 1FFFFFFFFFF4CFFFFFFFFFFFF 1FFFFFFFFFF4CFFFFFFFFFFFF 1FFFFFFFFFF4DFFFFFFFFFFFF 1FFFFFFFFFF4DFFFFFFFFFFFF 1FFFFFFFFFF4EFFFFFFFFFFFF 1FFFFFFFFFF4EFFFFFFFFFFFF 1FFFFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF4FFFFFFFFFFFFF 1FFFFFFFFFF50FFFFFFFFFFFF 1FFFFFFFFFF50FFFFFFFFFFFF 1FFFFFFFFFF51FFFFFFFFFFFF 1FFFFFFFFFF51FFFFFFFFFFFF 1FFFFFFFFFF52FFFFFFFFFFFF 1FFFFFFFFFF52FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF53FFFFFFFFFFFF 1FFFFFFFFFF54FFFFFFFFFFFF 1FFFFFFFFFF54FFFFFFFFFFFF 1FFFFFFFFFF55FFFFFFFFFFFF 1FFFFFFFFFF55FFFFFFFFFFFF 1FFFFFFFFFF56FFFFFFFFFFFF 1FFFFFFFFFF56FFFFFFFFFFFF 1FFFFFFFFFF57FFFFFFFFFFFF 1FFFFFFFFFF57FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF58FFFFFFFFFFFF 1FFFFFFFFFF59FFFFFFFFFFFF 1FFFFFFFFFF59FFFFFFFFFFFF 1FFFFFFFFFF5AFFFFFFFFFFFF 1FFFFFFFFFF5AFFFFFFFFFFFF 1FFFFFFFFFF5BFFFFFFFFFFFF 1FFFFFFFFFF5BFFFFFFFFFFFF 1FFFFFFFFFF5CFFFFFFFFFFFF 1FFFFFFFFFF5CFFFFFFFFFFFF 1FFFFFFFFFF5CFFFFFFFFFFFF 1FFFFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF5DFFFFFFFFFFFF 1FFFFFFFFFF5EFFFFFFFFFFFF 1FFFFFFFFFF5EFFFFFFFFFFFF 1FFFFFFFFFF5FFFFFFFFFFFFF 1FFFFFFFFFF5FFFFFFFFFFFFF 1FFFFFFFFFF60FFFFFFFFFFFF 1FFFFFFFFFF60FFFFFFFFFFFF 1FFFFFFFFFF61FFFFFFFFFFFF 1FFFFFFFFFF61FFFFFFFFFFFF 1FFFFFFFFFF62FFFFFFFFFFFF 1FFFFFFFFFF62FFFFFFFFFFFF 1FFFFFFFFFF63FFFFFFFFFFFF 1FFFFFFFFFF63FFFFFFFFFFFF 1FFFFFFFFFF64FFFFFFFFFFFF 1FFFFFFFFFF64FFFFFFFFFFFF 1FFFFFFFFFF65FFFFFFFFFFFF 1FFFFFFFFFF65FFFFFFFFFFFF 1FFFFFFFFFF66FFFFFFFFFFFF 1FFFFFFFFFF66FFFFFFFFFFFF 1FFFFFFFFFF67FFFFFFFFFFFF 1FFFFFFFFFF67FFFFFFFFFFFF 1FFFFFFFFFF68FFFFFFFFFFFF 1FFFFFFFFFF68FFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF69FFFFFFFFFFFF 1FFFFFFFFFF6AFFFFFFFFFFFF 1FFFFFFFFFF6AFFFFFFFFFFFF 1FFFFFFFFFF6BFFFFFFFFFFFF 1FFFFFFFFFF6BFFFFFFFFFFFF 1FFFFFFFFFF6CFFFFFFFFFFFF 1FFFFFFFFFF6CFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF70FFFFFFFFFFFF 1FFFFFFFFFF70FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF72FFFFFFFFFFFF 1FFFFFFFFFF72FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF77FFFFFFFFFFFF 1FFFFFFFFFF77FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF79FFFFFFFFFFFF 1FFFFFFFFFF79FFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7DFFFFFFFFFFFF 1FFFFFFFFFF7DFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFF7FFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7EFFFFFFFFFFFF 1FFFFFFFFFF7DFFFFFFFFFFFF 1FFFFFFFFFF7DFFFFFFFFFFFF 1FFFFFFFFFF7DFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7CFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7BFFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF7AFFFFFFFFFFFF 1FFFFFFFFFF79FFFFFFFFFFFF 1FFFFFFFFFF79FFFFFFFFFFFF 1FFFFFFFFFF79FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF78FFFFFFFFFFFF 1FFFFFFFFFF77FFFFFFFFFFFF 1FFFFFFFFFF77FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF76FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF75FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF74FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF73FFFFFFFFFFFF 1FFFFFFFFFF72FFFFFFFFFFFF 1FFFFFFFFFF72FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF71FFFFFFFFFFFF 1FFFFFFFFFF70FFFFFFFFFFFF 1FFFFFFFFFF70FFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF6FFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6EFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6DFFFFFFFFFFFF 1FFFFFFFFFF6CFFFFFFFFFFFF 1FFFFFFFFFF53''))');
INSERT INTO "SDE"."SDER" VALUES ('3', '0', '1765.195278', '281.077282', NULL, '0', '0', '0', '0', NULL, '0', '0', NULL, NULL, '中关村', '"SDE"."ST_GEOMETRY"(8,12,-295.122944192288,124.10766744218,-229.87916732303,161.443701604356,NULL,NULL,NULL,NULL,1765.19527764624,281.077281666722,300015,HEXTORAW(''FF 2 4 0 1 0 0 0FFFFFFFFFFFFFF 2FFFFFFFFFFFFFF FFFFFFFFFFFFF54FFFFFFFFFFFFFF 4FFFFFFFFFFFFFF 8FFFFFFFFFFFF13FFFFFFFFFFFF54FFFFFFFFFFFFFF 4FFFFFFFFFFFFFF 5FFFFFFFFFFFF 6FFFFFFFFFFFFFF 2FFFFFFFFFFFF 6FFFFFFFFFFFFFF 2FFFFFFFFFFFFFF 1FFFFFFFFFFFFFF 1FFFFFFFFFFFF 6FFFFFFFFFFFFFF 2FFFFFFFFFFFF 6FFFFFFFFFFFF3AFFFFFFFFFFFFFF 2FFFFFFFFFFFFFF 3FFFFFFFFFFFF 6FFFFFFFFFFFF2DFFFFFFFFFFFFFF 2''))');
INSERT INTO "SDE"."SDER" VALUES ('4', '0', '2287.731138', '292.644068', '324ewr23(', '0', '0', '0', '186489', NULL, '5612.521', '4854513.564', NULL, TO_TIMESTAMP('2018-09-13 00:00:00.000000', 'SYYYY-MM-DD HH24:MI:SS:FF6'), '123152eds', '"SDE"."ST_GEOMETRY"(264,11,-156.964983660037,51.8760999525154,-77.5995318160871,111.67964465183,NULL,NULL,NULL,NULL,2287.73113816301,292.644068371915,300015,HEXTORAW(''FF 2 4 0 1 0 0 0FFFFFFFFFFFFFF14FFFFFFFFFFFF4FFFFFFFFFFFFFFF 5 0 0FFFFFFFFFFFFFF 4FFFFFFFFFFFFFF 5 0 0FFFFFFFFFFFFFF 4FFFFFFFFFFFFFF14FFFFFFFFFFFF4FFFFFFFFFFFFFFF17FFFFFFFFFFFFFF 6FFFFFFFFFFFFFF 3FFFFFFFFFFFFFF 1FFFFFFFFFFFFFF 3FFFFFFFFFFFF30FFFFFFFFFFFFFF 1FFFFFFFFFFFFFF 4FFFFFFFFFFFFFF 6FFFFFFFFFFFFFF 2''))');

-- ----------------------------
-- Checks structure for table SDER
-- ----------------------------
ALTER TABLE "SDE"."SDER" ADD CONSTRAINT "SYS_C00486425" CHECK ("OBJECTID" IS NOT NULL) NOT DEFERRABLE INITIALLY IMMEDIATE NORELY VALIDATE;

-- ----------------------------
-- Indexes structure for table SDER
-- ----------------------------
CREATE UNIQUE INDEX "SDE"."R499_SDE_ROWID_UK"
  ON "SDE"."SDER" ("OBJECTID" ASC)
  NOLOGGING
  VISIBLE

INITRANS 4
STORAGE (
  INITIAL 65536 
  NEXT 1048576 
  MINEXTENTS 1
  MAXEXTENTS 2147483645
  BUFFER_POOL DEFAULT
  FLASH_CACHE DEFAULT
)
   USABLE;

```

# 158  get column_name from postgresql

for postgresql

```
SELECT column_name,data_type FROM information_schema.columns 
where table_name = 'mpf3';

COPY postgres.mpf80 FROM STDIN WITH BINARY;

```

for oracle
```
select column_name,data_type,data_length,data_precision,data_scale,nullable from all_tab_columns where table_name = 'SDEL' and owner = 'SDE' order by column_id

select a.SHAPE.SRID from SDE.SDER a

select a.FALSEX, a.FALSEY, a.xyunits, a.XYCLUSTER_TOL from sde.SPATIAL_REFERENCES a where a.auth_srid = 300015

SELECT OBJECTID,  a.SHAPE.points  ,  a.SHAPE.numpts ,a.SHAPE.SRID from SDE.SDER a  order by OBJECTID asc 

SELECT  g.OBJECTID, g.SHAPE.points, g.SHAPE.numpts, g.SHAPE.entity, g.SHAPE.minx, g.SHAPE.miny, g.SHAPE.maxx, g.SHAPE.maxy,g.SHAPE.SRID, g.rowid FROM SDE.road g ORDER BY OBJECTID


UPDATE SDE.road g SET a.SHAPE.points=NULL, g.SHAPE.numpts=0,g.SHAPE.entity=0, g.SHAPE.minx=0, g.SHAPE.miny=0, g.SHAPE.maxx=0, g.SHAPE.maxy=0, g.SHAPE.SRID = NULL WHERE g.OBJECTID = 2

SELECT OBJECTID,a.SHAPE  from SDE.T1 a order by OBJECTID asc

SELECT  g.OBJECTID, g.SHAPE.points, g.SHAPE.numpts, g.SHAPE.entity, g.SHAPE.minx, g.SHAPE.miny, g.SHAPE.maxx, g.SHAPE.maxy, g.rowid FROM SDE.T1 g ORDER BY OBJECTID


UPDATE SDE.T1 a SET a.SHAPE=NULL WHERE a.OBJECTID < 3


SELECT length(a.shape.points)  FROM SDE.SDER a 

SELECT max(length(a.shape.points))  FROM SDE.SDER a 

```


```
SELECT osm_id,way,ST_AsGeoJSON(way), ST_AsEWKT(way),ST_AsKML(way) FROM  public.planet_osm_line
```



# 159  oracle download url


```

https://download.oracle.com/otn_software/nt/instantclient/19800/instantclient-basic-windows.x64-19.8.0.0.0dbru.zip

https://download.oracle.com/otn_software/nt/instantclient/1919000/instantclient-basic-windows.x64-19.19.0.0.0dbru.zip
```

```
https://download.oracle.com/otn_software/nt/instantclient/1919000/instantclient-sdk-windows.x64-19.19.0.0.0dbru.zip

https://download.oracle.com/otn_software/linux/instantclient/1919000/instantclient-sdk-linux.x64-19.19.0.0.0dbru.zip

```
```

https://download.oracle.com/otn_software/linux/instantclient/1919000/instantclient-odbc-linux.x64-19.19.0.0.0dbru.zip


https://download.oracle.com/otn_software/nt/instantclient/1919000/instantclient-odbc-windows.x64-19.19.0.0.0dbru.zip
```

oracle version release time
------------------
```

发布	     已发布                              	高级支持					         扩展支持
21c	         1年零12个月前（2021 年 8 月 13 日）     8个月后结束（2024 年 4 月 30 日）     不可用
19c（长期）	  4年前（2019 年 4 月 25 日）             8个月后结束（2024 年 4 月 30 日）    3年零8个月结束（2027 年 4 月 30 日）

18c	         5年前（2018 年 7 月 23 日）             2年前结束（2021 年 6 月 30 日）       不可用
12c 版本2	  6年前（2017 年 3 月 1 日）             结束于 1 年零4个月前（2022 年3月31)   不可用
12c 版本1（长期）10年前（2013 年 6 月 25 日）        5年前结束（2018年7月31日）            结束于 1 年前（2022 年 7 月 31 日）
```


# 160  executeQuery return  ResultSet Methods next() and setDataBuffer in oracle

for more information, please visit following url: 
https://docs.oracle.com/cd/B19306_01/appdev.102/b14294/reference027.htm#i1082533

or search 'Oracle C++ Call Interface Programmer’s Guide.pdf' with Bing to get book guide;

<<Oracle C++ Call Interface Programmer’s Guide.pdf>>

## ResultSet

A ResultSet provides access to a table of data generated by executing a Statement. Table rows are retrieved in sequence. Within a row, column values can be accessed in any order.

A ResultSet maintains a cursor pointing to its current row of data. Initially the cursor is positioned before the first row. The next method moves the cursor to the next row.

The getxxx() methods retrieve column values for the current row. You can retrieve values using the index number of the column. Columns are numbered beginning at 1. For the getxxx() methods, OCCI attempts to convert the underlying data to the specified C++ type and returns a C++ value. SQL types are mapped to C++ types with the ResultSet::getxxx() methods.

## next()

******************************************************
ResultSet.next()  c++

This method fetches a specified number of rows, numRows, from a previously executed query, and reports the Status of this fetch as definded in Table 12-37.

For non-streamed mode, next() will only return the status of DATA_AVAILABLE or END_OF_FETCH.

When fetching one row at a time (numRows=1), process the data using getxxx() methods.

When fetching several rows at once (numRows>1), as in an Array Fetch, you must use the setDataBuffer() method to specify the location of your preallocated buffers before invoking next().

Up to numRows data records would populate the buffers specified by the setDataBuffer() call. To determine exactly how many records were returned, use the getNumArrayRows() method.


而需要注意的问题：

当获得一个结果集后，尤其是结果集只有一条记录时不要忘记调用next()来指向“下一条可用的”记录。如果不执行next()便访问结果集会报如下错误：

ORA-32129: cannot get information about this column

**************************************************

java.sql.ResultSet.next()
***********************************************
boolean java.sql.ResultSet.next() throws SQLException

Moves the cursor forward one row from its current position. A ResultSet cursor is initially positioned before the first row; the first call to the method next makes the first row the current row; the second call makes the second row the current row, and so on.

When a call to the next method returns false, the cursor is positioned after the last row. Any invocation of a ResultSet method which requires a current row will result in a SQLException being thrown. If the result set type is TYPE_FORWARD_ONLY, it is vendor specified whether their JDBC driver implementation will return false or throw anSQLException on a subsequent call to next.

If an input stream is open for the current row, a call to the method next will implicitly close it. A ResultSet object's warning chain is cleared when a new row is read.

ResultSet接口和方法next()的作用？
1、ResultSet表示数据库结果集的数据表，通常通过执行查询数据库的语句生成。

2、ResultSet对象保持一个光标指向其当前的数据行。 最初光标位于第一行之前。 next方法将光标移动到下一行，并且由于在ResultSet对象中没有更多行时返回false ，因此可以在while循环中使用循环来遍历结果集。

3、next()将光标从当前位置向前移动一行。 ResultSet光标最初位于第一行之前; 第一次调用方法next使第一行成为当前行; 第二次调用使第二行成为当前行，依此类推。 当调用next方法返回false时，光标位于最后一行之后。 任何调用需要当前行的ResultSet方法将导致抛出SQLException 。

4、结果 ：true如果新的当前行有效; false如果没有更多的行
***********************************************

## next的时候才与服务器交互取数据
使用occi编程，在进行大数据量的查询时，用到
              resultSet = hStmt->exeQuery(...)
请问，系统是一次性将所有查询到的记录都取出来放在resultSet，并分配空间的吗？如果是这样的话，大数据量的查询（查询结果可能几十万条）怎么控制内存空间的分配呢？


系统不是将数据全部取出来的，每次调用next的时候才与服务器交互取数据
可以用setPrefetchRowCount),设置预先读取的记录数
不过大量的数据查询建议你还是用setDataBuffer,不然效率很低的


哦，谢谢！
我想要先执行execute，然后一部分一部分（时间不确定）地从resultSet中用next取数据，这样能行吗？另外，setPrefetchRowCount或setDataBuffer是在execute命令之前还是之后执行，这个问题我一直很困惑。
非常感谢！

##  ORA-32104
ORA-32104

需要設置ORACLE_HOME

ORACLE_BASE=/u01/app/oracle
 ORACLE_HOME=/u01/app/oracle/product/10.2.0/DB_1
export ORACLE_BASE ORACLE_HOME 
***********************************************

## setDataBuffer()

对于ResultSet类中的next()方法，默认是一次检索一行数据，及一次检索执行一次网络往返，当结果集数量大时，效率低；对此OCCI提供了几种改善方法，即：在一次网络往返返回多行数据。

1. 通过使用setPrefetchRowCount()或setPrefetchMemorySize()方法设置预取属性
setPrefetchRowCount()设置要预取的行数，setPrefetchMemorySize()设置预取的大小。如果同时设置这两个属性，除非首先达到指定的内存限制，否则将预取指定的行数。如果首先达到指定的内存限制，则返回在调用setPreprichMemorySize()方法所定义的内存空间的尽可能多的行。默认情况下，预取属性是被打开的，并且数据库始终会获取一个额外的行。若要关闭预取属性，请将预取行计数和内存大小设置为0。

	void setPrefetchRowCount(unsigned int rowCount);
	void setPrefetchMemorySize(unsigned int bytes);

2. 通过setDataBuffer()方法提供特定缓冲区，为next()方法提供要返回的数据行数
对于前一种方法当返回数据后数据将存储在OCCI内置缓冲区中，我们使用setXXX()方法取出特定数据；当时setDataBuffer()方法提供特定缓冲区时，数据存储将存放在用户指定的缓冲区中。

注意：在调用next()方法前调用setDataBuffer()方法，使用getNumArrayRows()方法得到获取的数据行数，

	Example1.1

	ResultSet *resultSet = stmt->executeQuery(...); 

	resultSet->setDataBuffer(...); 

	while (resultSet->next(numRows) == DATA_AVAILABLE)  {
		process(resultSet->getNumArrayRows() );
	}


# 161 .   TOTP Algorithm: Reference Implementation


```
TOTP Algorithm: Reference Implementation

 <CODE BEGINS>

 /**
 Copyright (c) 2011 IETF Trust and the persons identified as
 authors of the code. All rights reserved.

 Redistribution and use in source and binary forms, with or without
 modification, is permitted pursuant to, and subject to the license
 terms contained in, the Simplified BSD License set forth in Section
 4.c of the IETF Trust's Legal Provisions Relating to IETF Documents
 (http://trustee.ietf.org/license-info).
 */

 import java.lang.reflect.UndeclaredThrowableException;
 import java.security.GeneralSecurityException;
 import java.text.DateFormat;
 import java.text.SimpleDateFormat;
 import java.util.Date;
 import javax.crypto.Mac;
 import javax.crypto.spec.SecretKeySpec;
 import java.math.BigInteger;
 import java.util.TimeZone;


 /**
  * This is an example implementation of the OATH
  * TOTP algorithm.
  * Visit www.openauthentication.org for more information.
  *
  * @author Johan Rydell, PortWise, Inc.
  */

 public class TOTP {

     private TOTP() {}

     /**
      * This method uses the JCE to provide the crypto algorithm.
      * HMAC computes a Hashed Message Authentication Code with the
      * crypto hash algorithm as a parameter.
      *
      * @param crypto: the crypto algorithm (HmacSHA1, HmacSHA256,
      *                             HmacSHA512)
      * @param keyBytes: the bytes to use for the HMAC key
      * @param text: the message or text to be authenticated
      */



M'Raihi, et al.               Informational                     [Page 9]

RFC 6238                      HOTPTimeBased                     May 2011


     private static byte[] hmac_sha(String crypto, byte[] keyBytes,
             byte[] text){
         try {
             Mac hmac;
             hmac = Mac.getInstance(crypto);
             SecretKeySpec macKey =
                 new SecretKeySpec(keyBytes, "RAW");
             hmac.init(macKey);
             return hmac.doFinal(text);
         } catch (GeneralSecurityException gse) {
             throw new UndeclaredThrowableException(gse);
         }
     }


     /**
      * This method converts a HEX string to Byte[]
      *
      * @param hex: the HEX string
      *
      * @return: a byte array
      */

     private static byte[] hexStr2Bytes(String hex){
         // Adding one byte to get the right conversion
         // Values starting with "0" can be converted
         byte[] bArray = new BigInteger("10" + hex,16).toByteArray();

         // Copy all the REAL bytes, not the "first"
         byte[] ret = new byte[bArray.length - 1];
         for (int i = 0; i < ret.length; i++)
             ret[i] = bArray[i+1];
         return ret;
     }

     private static final int[] DIGITS_POWER
     // 0 1  2   3    4     5      6       7        8
     = {1,10,100,1000,10000,100000,1000000,10000000,100000000 };



M'Raihi, et al.               Informational                    [Page 10]

RFC 6238                      HOTPTimeBased                     May 2011


     /**
      * This method generates a TOTP value for the given
      * set of parameters.
      *
      * @param key: the shared secret, HEX encoded
      * @param time: a value that reflects a time
      * @param returnDigits: number of digits to return
      *
      * @return: a numeric String in base 10 that includes
      *              {@link truncationDigits} digits
      */

     public static String generateTOTP(String key,
             String time,
             String returnDigits){
         return generateTOTP(key, time, returnDigits, "HmacSHA1");
     }


     /**
      * This method generates a TOTP value for the given
      * set of parameters.
      *
      * @param key: the shared secret, HEX encoded
      * @param time: a value that reflects a time
      * @param returnDigits: number of digits to return
      *
      * @return: a numeric String in base 10 that includes
      *              {@link truncationDigits} digits
      */

     public static String generateTOTP256(String key,
             String time,
             String returnDigits){
         return generateTOTP(key, time, returnDigits, "HmacSHA256");
     }


M'Raihi, et al.               Informational                    [Page 11]

RFC 6238                      HOTPTimeBased                     May 2011


     /**
      * This method generates a TOTP value for the given
      * set of parameters.
      *
      * @param key: the shared secret, HEX encoded
      * @param time: a value that reflects a time
      * @param returnDigits: number of digits to return
      *
      * @return: a numeric String in base 10 that includes
      *              {@link truncationDigits} digits
      */

     public static String generateTOTP512(String key,
             String time,
             String returnDigits){
         return generateTOTP(key, time, returnDigits, "HmacSHA512");
     }


     /**
      * This method generates a TOTP value for the given
      * set of parameters.
      *
      * @param key: the shared secret, HEX encoded
      * @param time: a value that reflects a time
      * @param returnDigits: number of digits to return
      * @param crypto: the crypto function to use
      *
      * @return: a numeric String in base 10 that includes
      *              {@link truncationDigits} digits
      */

     public static String generateTOTP(String key,
             String time,
             String returnDigits,
             String crypto){
         int codeDigits = Integer.decode(returnDigits).intValue();
         String result = null;

         // Using the counter
         // First 8 bytes are for the movingFactor
         // Compliant with base RFC 4226 (HOTP)
         while (time.length() < 16 )
             time = "0" + time;

         // Get the HEX in a Byte[]
         byte[] msg = hexStr2Bytes(time);
         byte[] k = hexStr2Bytes(key);



M'Raihi, et al.               Informational                    [Page 12]

RFC 6238                      HOTPTimeBased                     May 2011


         byte[] hash = hmac_sha(crypto, k, msg);

         // put selected bytes into result int
         int offset = hash[hash.length - 1] & 0xf;

         int binary =
             ((hash[offset] & 0x7f) << 24) |
             ((hash[offset + 1] & 0xff) << 16) |
             ((hash[offset + 2] & 0xff) << 8) |
             (hash[offset + 3] & 0xff);

         int otp = binary % DIGITS_POWER[codeDigits];

         result = Integer.toString(otp);
         while (result.length() < codeDigits) {
             result = "0" + result;
         }
         return result;
     }

     public static void main(String[] args) {
         // Seed for HMAC-SHA1 - 20 bytes
         String seed = "3132333435363738393031323334353637383930";
         // Seed for HMAC-SHA256 - 32 bytes
         String seed32 = "3132333435363738393031323334353637383930" +
         "313233343536373839303132";
         // Seed for HMAC-SHA512 - 64 bytes
         String seed64 = "3132333435363738393031323334353637383930" +
         "3132333435363738393031323334353637383930" +
         "3132333435363738393031323334353637383930" +
         "31323334";
         long T0 = 0;
         long X = 30;
         long testTime[] = {59L, 1111111109L, 1111111111L,
                 1234567890L, 2000000000L, 20000000000L};

         String steps = "0";
         DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
         df.setTimeZone(TimeZone.getTimeZone("UTC"));



M'Raihi, et al.               Informational                    [Page 13]

RFC 6238                      HOTPTimeBased                     May 2011


         try {
             System.out.println(
                     "+---------------+-----------------------+" +
             "------------------+--------+--------+");
             System.out.println(
                     "|  Time(sec)    |   Time (UTC format)   " +
             "| Value of T(Hex)  |  TOTP  | Mode   |");
             System.out.println(
                     "+---------------+-----------------------+" +
             "------------------+--------+--------+");

             for (int i=0; i<testTime.length; i++) {
                 long T = (testTime[i] - T0)/X;
                 steps = Long.toHexString(T).toUpperCase();
                 while (steps.length() < 16) steps = "0" + steps;
                 String fmtTime = String.format("%1$-11s", testTime[i]);
                 String utcTime = df.format(new Date(testTime[i]*1000));
                 System.out.print("|  " + fmtTime + "  |  " + utcTime +
                         "  | " + steps + " |");
                 System.out.println(generateTOTP(seed, steps, "8",
                 "HmacSHA1") + "| SHA1   |");
                 System.out.print("|  " + fmtTime + "  |  " + utcTime +
                         "  | " + steps + " |");
                 System.out.println(generateTOTP(seed32, steps, "8",
                 "HmacSHA256") + "| SHA256 |");
                 System.out.print("|  " + fmtTime + "  |  " + utcTime +
                         "  | " + steps + " |");
                 System.out.println(generateTOTP(seed64, steps, "8",
                 "HmacSHA512") + "| SHA512 |");

                 System.out.println(
                         "+---------------+-----------------------+" +
                 "------------------+--------+--------+");
             }
         }catch (final Exception e){
             System.out.println("Error : " + e);
         }
     }
 }

<CODE ENDS>

```

# 162  .  sscanf(strInfo, "%hhd;%hhd;%ld;%ld;%s",)

##  %hhd;%hhd;%ld;%ld;%s
```
#pragma pack(push,1)
typedef	struct	SVCINFO
{
	char		svcName[512];	//服务名称
	char		dnsName[512];	//对应的服务名
	char		svcType;					//服务类型
	
	DWORD		ipAdd;						//IP地址
	char		ptcType;					//协议类型
	long		portNo;						//端口号
#ifdef __cplusplus
	SVCINFO()							
	{
		svcName[0]= 0;
		dnsName[0]= 0;
		svcType	  = 0;
		ipAdd	  = 0;
		ptcType   = 0;
		portNo    = 0; 
	}
#endif
	
}SVCINFO;
#pragma pack(pop)

/////////

SVCINFO info;
SVCINFO* pInfo = &info;
char szInfo[1024] = "\0";
sprintf(szInfo, "%hhd;%hhd;%ld;%ld;%s",
	pInfo->ptcType, pInfo->svcType, pInfo->ipAdd, pInfo->portNo, pInfo->dnsName);

std::string strInfo = "0;21;-1;5432;localhost:5432/osm"
sscanf(strInfo.c_str(), "%hhd;%hhd;%ld;%ld;%s",
		&info.ptcType, &info.svcType, &info.ipAdd, &info.portNo, info.dnsName);
```

```
不同转换规范代表的转换方式如下表：

长度指示符	转换规范	转换为某种类型的二进制
hh	d	char
h	d	short int
无	d	int
l	d	long
ll	d	long long
hh	u	unsigned char
h	u	unsigned short int
无	u	unsigned int
l	u	unsigned long
ll	u	unsigned long long
无	f	float
l	f	double
无	c	char
无	s	char


根据上表，对照我们的例子中的转换方式如下。

子串"1"对应转换规范"%hhd"，将转换为char类型的二进制表示，1字节。

子串"2"对应转换规范"%hd"，将转换为short类型的二进制表示，2字节。

子串"3"对应转换规范"%d"，将转换为int类型的二进制表示，4字节。

子串"4"对应转换规范"%ld"，将转换为long类型的二进制表示，4字节。

子串"5.6"对应转换规范"%f"，将转换为float类型的二进制表示，4字节。

子串"7.8"对应转换规范"%lf"，将转换为double类型的二进制表示，8字节
```

##  sscanf
C 库函数 int sscanf(const char *str, const char *format, ...) 从字符串读取格式化输入
如果成功，该函数返回成功匹配和赋值的个数。如果到达文件末尾或发生读错误，则返回 EOF。
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main()
{
   int day, year;
   char weekday[20], month[20], dtm[100];

   strcpy( dtm, "Saturday March 25 1989" );
   sscanf( dtm, "%s %s %d  %d", weekday, month, &day, &year );

   printf("%s %d, %d = %s\n", month, day, year, weekday );
    
   return(0);
}
```
让我们编译并运行上面的程序，这将产生以下结果：
```
March 25, 1989 = Saturday
```

# 163  .  auto send email with python 

```
# auto_send_mail.py
#from smtplib import SMTP

import os
import smtplib
from email import encoders
from email.header import Header
from email.mime.image import MIMEImage
from email.mime.text import MIMEText
from email.mime.base import MIMEBase
from email.mime.multipart import MIMEMultipart
from email.mime.application import MIMEApplication
from email.utils import parseaddr, formataddr
from email.utils import make_msgid

def auto_send_mail() :
	# 请自行修改下面的邮件发送者和接收者
	#【发送者邮箱】
	sender = 'chelly@outlook.com'
	#【接受者邮箱】
	to_list = 'chelly@outlook.com'
	#cc_list = ['ycaa@outlook.com', 'chelly@outlook.com']
	cc_list = 'ycaa@outlook.com'
	#【邮件内容】
	message =  MIMEMultipart('related')
	#message = MIMEText('hello world ! now i can send email with python modudle.', 'plain', 'utf-8')
	#【发件人】
	message['From'] = 'cycle'
	#【收件人】
	#message['To'] = Header('chelly@outlook.com')
	message['To'] = 'chelly@outlook.com'
	#【邮件主题】
	message['Subject'] = Header('Butter Cookies License', 'utf-8')
	#message['Message-ID'] = make_msgid()
	#---这是附件部分---
	#xlsx类型附件
	#part = MIMEApplication(open(r'./file/foo.xlsx','rb').read())
	#part.add_header('Content-Disposition', 'attachment', filename="foo.xlsx")
	#message.attach(part)

	##pdf类型附件
	part = MIMEApplication(open(r'F:/User/Pictures/P020230210433055525061.pdf','rb').read())
	part.add_header('Content-Disposition', 'attachment', filename="P020230210433055525061.pdf")
	message.attach(part)
	
	##py类型附件,会被认为是垃圾邮件
	#part = MIMEApplication(open(r'F:/User/Desktop/auto_send_mail.py','rb').read())
	#part.add_header('Content-Disposition', 'attachment', filename="auto_send_mail.py")
	#message.attach(part)
	
	# 添加附件就是加上一个MIMEBase，从本地读取一个图片:
	with open('F:/User/Pictures/github-f2a-recovery-codes.png', 'rb') as f:
		# 设置附件的MIME和文件名，这里是png类型:
		mime = MIMEBase('image', 'png', filename='github-f2a-recovery-codes.png')
		# 加上必要的头信息:
		mime.add_header('Content-Disposition', 'attachment', filename='github-f2a-recovery-codes.png')
		mime.add_header('Content-ID', '<0>')
		mime.add_header('X-Attachment-Id', '0')
		# 把附件的内容读进来:
		mime.set_payload(f.read())
		# 用Base64编码:
		encoders.encode_base64(mime)
		# 添加到MIMEMultipart:
		message.attach(mime)
		# 然后，按正常发送流程把msg（注意类型已变为MIMEMultipart）发送出去，就可以
	
	# 正文展示图片和一个链接
	
# plain
	plain_text_head = """
		<p>Hello,cycle</p> 
		<p>The license key for your Butter cookies trial is ready to be installed.</p>
		<p>This is one of two mails you will receive with unique license keys.  Deploy the Butter cookies Virtual Appliance you downloaded at each end of the WAN link you plan to optimize. </p>
		<p>Once deployed, install the unique license key below on one appliance and the unique license key from the second email on the other appliance to enable optimization.</p> 
		<p></p>"""
		
	plain_text_license = """
		<p>Product: <b>ButterCookies Pro 2023</b></p>  
		<p>License Key:</p> 
		<p><b>P6F0-lBmL-34Pm-IZ3m-3MGL-s2ND-eLKC-+3pe-HROq-VMGQ-spH</b></p>
		<p><b>kOBb-LJOV-QC3p-nPw1-OJGy-KUem-egFc-sXWB-uDwm-goPa-gw==</b></p>
		<p>Your license will expire on: <b>Mon Jan 19 2023</b></p>
		<p></p>
		"""
		
	plain_text_tail = """
		<p>Before you start your trial, we recommend you visit the Butter cookies Marketplace website and read the ‘Getting Started’ page:</p> 
		<p>http://cyclezone.com/products/butter-cookies/2023.html</p> 
		<p></p>
		<p>If you have any questions about installing and configuring your virtual appliance, you can contact the Butter cookies support team directly.</p>  
		<p>The support team can be reached in one of three ways:</p>
		<p>Via the support center at http://www.cyclezone.com/support/portal_login.asp </p>
		<p>you should have received an email with instructions on how to enable your account</p>
		<p>By phone at <b>866-665-7325 / 865-655-1850</b></p> 
		<p>By email at <b>support@cyclezone.com</b></p>
		<p></p>
		"""
		
	plain_text_Logo = """
		<html><body><h1>github-f2a-codes</h1><p><img src="cid:0"></p></body></html>
		"""
		
	plain_text = """
		<p>Hello,cycle</p> 
		<p>The license key for your Butter cookies trial is ready to be installed.</p>
		<p>This is one of two mails you will receive with unique license keys.  Deploy the Butter cookies Virtual Appliance you downloaded at each end of the WAN link you plan to optimize. </p>
		<p>Once deployed, install the unique license key below on one appliance and the unique license key from the second email on the other appliance to enable optimization.</p> 
		<p></p>
		<p>Product: <b>ButterCookies Pro 2023</b></p>  
		<p>License Key:</p> 
		<p><b>P6F0-lBmL-34Pm-IZ3m-3MGL-s2ND-eLKC-+3pe-HROq-VMGQ-spH</b></p>
		<p><b>kOBb-LJOV-QC3p-nPw1-OJGy-KUem-egFc-sXWB-uDwm-goPa-gw==</b></p>
		<p>Your license will expire on: <b>Mon Jan 19 2023</b></p>
		<p></p>
		<p>Before you start your trial, we recommend you visit the Butter cookies Marketplace website and read the ‘Getting Started’ page:</p> 
		<p>https://cyclezone.github.io</p> 
		<p></p>
		<p>If you have any questions about installing and configuring your virtual appliance, you can contact the Butter cookies support team directly.</p>  
		<p>The support team can be reached in one of three ways:</p>
		<p>Via the support center at http://www.cyclezone.com/support/portal_login.asp </p>
		<p>you should have received an email with instructions on how to enable your account</p>
		<p>By phone at <b>866-665-7325 / 865-655-1850</b></p> 
		<p>By email at <b>support@cyclezone.com</b></p>
		<p></p>
		<p>For more detail,please visit <a href="https://cyclezone.github.io">cyclezone</a></p>
		<html><body><h1>github-f2a-codes</h1><p><img src="cid:0"></p></body></html>"""

	##<p>For more detail,please visit <a href="https://cyclezone.github.io">cyclezone</a></p>
	plain_context = plain_text_head + plain_text_license + plain_text_tail + plain_text_Logo
		
	plain = MIMEText(plain_context, 'html', 'utf-8')
	message.attach(plain)
	# 详细代码
	######### outlook.com #########
	# 发件人邮箱地址
	sendAddress = 'chelly@outlook.com'
	port_no = 587
	email_host = 'smtp.office365.com'
	# 发件人授权码
	# password = 'xbbgckycdiylcsgq'
	password = 'xnmckaezexajzkhq'
	print('begin connect: ' + sendAddress)
	send_list = cc_list
	print('send to : ' + send_list)
	try :
	 # 连接服务器
	 # 登录邮箱
	 smtp = smtplib.SMTP(email_host, port_no)
	 #smtp.set_debuglevel(1)
	 smtp.connect(email_host, port_no)
	 smtp.ehlo()
	 smtp.starttls()
	 smtp.login(sender, password)
	 print('smtp.login.ok')
	 smtp.sendmail(sender, send_list, message.as_string())
	 smtp.noop()
	 smtp.quit()
	 print('smtp.sendmail.ok!')
	except:
	 print("email send failed")
 
	

#xnmckaezexajzkhq
##POP
#服务器名称: outlook.office365.com
#端口: 995
#加密方法: TLS

##IMAP
#服务器名称: outlook.office365.com
#端口: 993
#加密方法: TLS

##SMTP：
#服务器名称: smtp.office365.com
#端口: 587
#加密方法: STARTTLS

if __name__ == '__main__':
    auto_send_mail()


```

# 164  . create and destroy process lock in shared memory

```
///共享内存上使用进程锁

#include <unistd.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <assert.h>
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
/**
* 返回一片共享内存标识符，用于后续获取该共享内存，以及销毁该共享内存
* INDEX_OF_KEY —— 自定义的该共享内存序号
* LENGTH —— 共享内存大小
*/
const int create_flag(const int INDEX_OF_KEY, const unsigned int LENGTH) {
	// 生成key
	const char* FILE_PATH = "./";
	key_t key = ftok(FILE_PATH, INDEX_OF_KEY);
	// 创建共享内存空间
	const int FLAG = shmget(key, LENGTH, IPC_CREAT | 0666);
	return FLAG;
}
// 定义进程锁结构体
typedef struct MUTEX_PACKAGE {
	// 锁以及状态
	pthread_mutex_t lock;
	pthread_mutexattr_t lock_attr;
	// 在共享内存中的标识符
	int FLAG;
} mutex_package_t;
// 初始化进程锁结构体
const int init(void* pthis) {
	mutex_package_t* mp = (mutex_package_t*)pthis;
	// 初始化锁状态，设置状态状态为——进程共享
	pthread_mutexattr_init(&(mp->lock_attr));
	pthread_mutexattr_setpshared(&(mp->lock_attr), PTHREAD_PROCESS_SHARED);
	// 用锁状态来初始化锁
	pthread_mutex_init(&(mp->lock), &(mp->lock_attr));
	return 0;
}
// 在共享内存上定义进程锁结构体并且返回其位置
mutex_package_t* create_mutex_package(const int INDEX) {
	const int FLAG = create_flag(INDEX, sizeof(mutex_package_t));
	mutex_package_t* mp = (mutex_package_t*)shmat(FLAG, NULL, SHM_R | SHM_W);
	mp->FLAG = FLAG;
	assert(init(mp) == 0);
	return mp;
}
// 销毁进程锁结构体，利用其FLAG变量索引到其占用的共享内存并销毁
const int destory_mutex_package(mutex_package_t* mp) {
	// 销毁锁和锁状态
	pthread_mutex_destroy(&(mp->lock));
	pthread_mutexattr_destroy(&(mp->lock_attr));
	// 释放共享内存
	assert(shmctl(mp->FLAG, IPC_RMID, NULL) == 0);
	return 0;
}
int main() {
	// 创建自定义进程锁
	mutex_package_t* mp = create_mutex_package(111);
	// 获取一片共享内存空间
	const int FLAG = create_flag(222, sizeof(int));
	volatile int* x = (int*)shmat(FLAG, NULL, 0);

	// 创建新进程
	int id = fork();
	assert(id >= 0);
	// 设置循环次数
	const int N = 1000000;
	// 父进程每次加1，子进程每次加2
	int i;
	for (i = 0; i < N; ++i) {
		if (id > 0) { // 父进程
					  // 加锁
			pthread_mutex_lock(&(mp->lock));
			int temp = *x;
			*x = temp + 1;
			// 解锁
			pthread_mutex_unlock(&(mp->lock));
		}
		else { // 子进程
			   // 加锁
			pthread_mutex_lock(&(mp->lock));
			int temp = *x;
			*x = temp + 2;
			// 解锁
			pthread_mutex_unlock(&(mp->lock));
		}
	}
	// 等待循环完毕
	sleep(1);
	// 打印
	printf("pid= %d, x_address= %x, x= %d\n", getpid(), x, *x);
	// 等待打印完毕
	sleep(1);
	// 销毁进程锁,释放申请的共享内存
	if (id > 0) { // 父进程
		destory_mutex_package(mp);
		mp = NULL;
		shmctl(FLAG, IPC_RMID, NULL);
		x = NULL;
		printf("父进程释放资源完毕\n");
	}
	return 0;
}
```
# 165  . start process in python

```

# 第一种创建进程的方式
from multiprocessing import Process
import time


def task(name):
    print('%s is running' % name)
    time.sleep(3)
    print('%s is over' % name)


if __name__ == '__main__':
    p = Process(target=task, args=('jason',))  # 创建一个进程对象
    p.start()  # 告诉操作系统创建一个新的进程
    print('主进程')

# 创建进程的第二种方式
from multiprocessing import Process
import time
class MyProcess(Process):
    def __init__(self, username):
        self.username = username
        super().__init__()
    def run(self):
        print('你好啊 小姐姐',self.username)
        time.sleep(3)
        print('get out!!!',self.username)
if __name__ == '__main__':
    p = MyProcess('tony')
    p.start()
    print('主进程')
    
该段代码就是主进程，因为用模块启动了新的子进程，所以在内存空间中新申请一块内存空间用于存放子进程代码，


```

```
from multiprocessing import Process
import time


def task(name, n):
    print(f'{name} is running')
    time.sleep(n)
    print(f'{name} is over')


if __name__ == '__main__':
    p1 = Process(target=task, args=('jason', 1))
    p2 = Process(target=task, args=('tony', 2))
    p3 = Process(target=task, args=('kevin', 3))
    start_time = time.time()
    p1.start()
    p2.start()
    p3.start()
    p1.join()
    p2.join()
    p3.join()
    end_time = time.time() - start_time
    print('主进程', f'总耗时:{end_time}')  # 主进程 总耗时:3.015652894973755,在p1进行join时候，p2,p3也同时在进行，所以时间是最大的那个进程的耗时
    # 如果是一个start一个join交替执行 那么总耗时就是各个任务耗时总和


```

```
# 代码模拟抢票(有问题)
import json
from multiprocessing import Process
import time
import random
from multiprocessing import Process, Lock


# 查票
def search(name):
    with open(r'ticket_data.json', 'r', encoding='utf8') as f:
        data = json.load(f)
    print(f'{name}查询当前余票:%s' % data.get('ticket_num'))


# 买票
def buy(name):
    '''
    点击买票是需要再次查票的 因为期间其他人可能已经把票买走了
    '''
    # 1.查票
    with open(r'ticket_data.json', 'r', encoding='utf8') as f:
        data = json.load(f)
    time.sleep(random.randint(1, 3))
    # 2.判断是否还有余票
    if data.get('ticket_num') > 0:
        data['ticket_num'] -= 1
        with open(r'ticket_data.json', 'w', encoding='utf8') as f:
            json.dump(data, f)
        print(f'{name}抢票成功')
    else:
        print(f'{name}抢票失败 没有余票了')


def run(name):
    search(name)
	mutex.acquire()  # 抢锁
    buy(name)
	mutex.release()  # 放锁


# 模拟多人同时抢票
if __name__ == '__main__':
	mutex = Lock()
    for i in range(1, 10):
        p = Process(target=run, args=('用户:%s' % i,))
        p.start()
        

```

# 166 . UUID

UUID是指在一台机器上生成的数字，它保证对在同一时空中的所有机器都是唯一的。
通常平台会提供生成的API。
按照开放软件基金会(OSF)制定的标准计算，
用到了全局唯一的IEEE机器识别号、纳秒级时间、芯片ID码和许多可能的数字。
如果你在生成一个UUID之后，过几秒又生成一个UUID，
则第一个部分不同，其余相同(实际测试结果每次都不同）。
即每次生成的UUID都是不同的。

UUID由以下几部分的组合：

	1 全局唯一的IEEE机器识别号。（如果有网卡，从网卡MAC地址获得；没有网卡，以其他方式获得）
	2 纳秒级时间
	3 芯片ID码
	4 许多可能的数字

UUID的唯一缺陷在于生成的结果串会比较长。
关于UUID这个标准使用最普遍的是微软的GUID(Globals Unique Identifiers)。
在ColdFusion中可以用CreateUUID()函数很简单地生成UUID，

	其格式为：xxxxxxxx - xxxx - xxxx - xxxxxxxxxxxxxxxx(8 - 4 - 4 - 16)，

其中每个x是0 - 9 a - f 范围内的一个十六进制的数字。

	而标准的UUID格式为：xxxxxxxx - xxxx - xxxx - xxxx - xxxxxxxxxxxx(8 - 4 - 4 - 4 - 12)


```
#include <stdio.h>
#include <string>
#include <iostream>

#ifdef WIN32
#include <objbase.h>
#else
#include <uuid/uuid.h>
#endif

	using namespace std;

#define MAX_LEN 128


/*
**@brief: get windows guid or linux uuid
**@return: string type windows guid or linux uuid
*/
string GetGuid()
{
	char szuuid[MAX_LEN] = { 0 };
#ifdef WIN32
	GUID guid;
	CoCreateGuid(&guid);
	_snprintf_s(
		szuuid,
		sizeof(szuuid),
		"{%08X-%04X-%04X-%02X%02X-%02X%02X%02X%02X%02X%02X}",
		guid.Data1, guid.Data2, guid.Data3,
		guid.Data4[0], guid.Data4[1],
		guid.Data4[2], guid.Data4[3],
		guid.Data4[4], guid.Data4[5],
		guid.Data4[6], guid.Data4[7]);
#else
	uuid_t uuid;
	uuid_generate(uuid);
	uuid_unparse(uuid, szuuid);
#endif

	return std::string(szuuid);
}

int main()
{
	string strGuid = GetGuid();
	cout << strGuid.c_str() << endl;
	return 0;
}
```



# 167 . How to generate and check serial codes


## 1. 注册码 / 序列号组成

1. 产品版本：限定具体到哪个版本、同产品的其他版本不能用。
2. 到期时间：限定截止服务运行时间，到给定截止前一周会有界面提示和邮件提醒。
3. 唯一标识：限定一个软件产品和MAC地址或者磁盘UUID(linux系统) / GUID（Windows系统）绑定。
4. 预留字段：用于分模块限定核心功能、分模块收费等。

以下是Silver - peak产品序列号使用期限截止前一周前邮件提示全文：
```
Hello,
This is to notify you that the Proof of Concept currently in process at XXX@163.com will expire on Wed Jan 21 2015.
This affects the following appliances, shipped on Mon Dec 22 2014:
VX - 2000, S / N 001BBC03A8E9
VX - 2000, S / N 001BBC03A8EA
If you have questions, please contact your account executive(Tricia Png, tpng@silver-peak.com).
Thank you,
Silver Peak Systems

```

## 2. 序列号生成逻辑

1.  获取安装设备的UUID，如：
	9d669361 - 7f8a - 4f97 - b08a - 488e4a92ee52；
	该UUID应该存储在设备的软件安装路径一份，以备对比验证。
2.  填写对应安装的软件版本号，如1.0.0.1；
3.  填写使用或授权限定使用的期限，如3年。
4.  点击生成序列号生成授权序列号（后台会调用RSA加密算法，对输入内容进行加密）。


## 3. 序列号验证逻辑

1. 输入获取的序列号。
2. 后台执行RSA解密序列号。
3. 判定各个属性值和安装设备是否一致。
4. 全部相同确定为有效序列号，可以放行软件功能权限。


# 168  print error msg 

```
size_t sde_err(std::string& szerr, std::string& szSql, int code,const char* func,int ln)
{
	std::string stFile = __FILE__;
	int pos = stFile.rfind("\\");

	char szError[2048] = "\0";
	sprintf(szError, "| code:[%d] ,%s(%d): %s ", code, stFile.substr(pos + 1).c_str(), ln, func);
	szerr = szerr  + szError + "| " + szSql;

	Log("sde_error", "%s", szerr.c_str());
	return szerr.size();
}

#define  error(ex,sql,code) sde_err(ex, sql, code, __FUNCTION__, __LINE__);

//////demo

try{}
catch (SQLException &sqlExcp)
{
	int i = sqlExcp.getErrorCode();
	string strinfo = sqlExcp.getMessage(); 
	error(strinfo,sql, i);
}

```

# 169 Auto Oracle Query


```
typedef struct tagAutoOracleQuery
{
	ResultSet * pRs;
	Statement * pStmt;
	Connection *pConn;
	COracleDataBase *pGDB;
	std::string	sql;

	tagAutoOracleQuery(COracleDataBase *gdb) :pRs(nullptr),
		pStmt(nullptr),
		pConn(nullptr),
		pGDB(gdb)
	{
		Initialize(pGDB);
	}

	~tagAutoOracleQuery()
	{
		DeInitialize();
	}

	ResultSet * Execute(std::string& strSQL)
	{
		if (pStmt == nullptr) return nullptr;
		pRs = pStmt->executeQuery(strSQL);
		return pRs;
	}

private:
	int Initialize(COracleDataBase *pGDB)
	{
		if (pGDB == nullptr) return -1;
		pConn = pGDB->AcquireConn();
		if (pConn == nullptr) return -2;
		pStmt = pConn->createStatement();
		if (pStmt == nullptr) return -4;
		return 1;
	}

	void DeInitialize()
	{
		if (pStmt != nullptr)
		{
			if (pRs != nullptr)
			{
				pStmt->closeResultSet(pRs);
				pRs = nullptr;
			}
			pConn->terminateStatement(pStmt);
			pStmt = nullptr;
		}
		if (pGDB && pConn != nullptr)
		{
			pGDB->ReleaseConn(pConn);
			pConn = nullptr;
		}
	}

}AutoOracleQuery;

#define throw_exception_sql(sqlExcp,strSQL)     \
		int i = sqlExcp.getErrorCode();         \
		string strinfo = sqlExcp.getMessage();  \
		error(strinfo, strSQL, i);              \
		SetLastError(ERR_IO_FAIL);             \
		SetLastAuxiliaryErrorMsg(strinfo.c_str());


long GetNullObjCount()
{
	//"SELECT OBJECTID   from SDE.T1 a WHERE a.SHAPE IS  NULL order by OBJECTID asc"
	//	"SELECT COUNT(OBJECTID)   from SDE.T1 a WHERE a.SHAPE IS  NULL";

	char szSQL[1024] = { 0 };
	sprintf(szSQL, "SELECT COUNT(%s) FROM  %s  a  ", strObjectName.c_str(), xclsInf.name);
	std::string	strSQL = szSQL;
	strSQL += shape_pts_is_null;
	ResultSet * pRs = nullptr;
	long ncount = 0;

	try
	{
		AutoOracleQuery query(m_ptGDB);
		pRs = query.Execute(strSQL);
		if (pRs&&pRs->next())
			ncount = pRs->getInt(1);
	}
	catch (SQLException &sqlExcp)
	{
		throw_exception_sql(sqlExcp, strSQL);
	}
	return ncount;
}

long GetNullObjIDs()
{
	//"SELECT OBJECTID   from SDE.T1 a WHERE a.SHAPE IS  NULL order by OBJECTID asc"
	//	"SELECT COUNT(OBJECTID)   from SDE.T1 a WHERE a.SHAPE IS  NULL";
	char szSQL[1024] = { 0 };
	sprintf(szSQL, "SELECT %s FROM  %s  a  ", strObjectName.c_str(), xclsInf.name);
	std::string	strSQL = szSQL;
	strSQL += shape_pts_is_null;
	sprintf(szSQL, " ORDER BY %s asc  ", strObjectName.c_str());
	strSQL += szSQL;
	ResultSet * pRs = nullptr;
	long ncount = 0;
	m_objIds.clear();
	try
	{
		AutoOracleQuery query(m_ptGDB);
		pRs = query.Execute(strSQL);
		const int num = 500;
		while (pRs&&pRs->next()){
			int id = pRs->getInt(1);
			m_objIds.push_back(id);
			if(m_objIds.size() > num + 2) break;
		}
	}
	catch (SQLException &sqlExcp)
	{
		throw_exception_sql(sqlExcp, strSQL);
	}
	return ncount;

}

long CheckNullObj()
{
	if (m_begin == 0) return 0;
	const char* szTag = "sde_null";
	
	long ncount = GetNullObjCount();
	if (ncount > 0) 
	{
		yLog(szTag, "sde null object count：%d", ncount);
		GetNullObjIDs();

		char szVal[256] = "\0";
		sprintf(szVal, "(%s, %s)", xclsInf.owner, xclsInf.name);
		std::string str(szVal);
		str += "无效几何数据OBJECTID";
		size_t nsize = m_objIds.size();
		const int num = 500;
		if (nsize > num) {
			sprintf(szVal, " [%d/%d]", num, nsize);
			str += szVal;
		}
		str += ":";
		if (nsize > num) nsize = num;
		for (size_t m = 0; m < nsize; m++)
		{
			sprintf(szVal, "%d,", m_objIds[m]);
			str += szVal;
		}
		yLog(szTag, "%s", str.c_str());
		SetLastError(ERR_GEOMETY_CORRUPT_STREAM);
		SetLastAuxiliaryErrorMsg(str.c_str());
	}
	return ncount;
}

```
# 170 Oracle server connect

```
#define  SERVER_CONNECT_DEFAULT_MIN_POOLSIZE  5
#define  SERVER_CONNECT_DEFAULT_MAX_POOLSIZE  10
#define  SERVER_CONNECT_INCREAMENT_POOLSIZE   5
class CORLServer
{
public:
	CORLServer();

	~CORLServer();

public:
	string					m_Svr;		//服务名(Oracle实例名)
	string                  m_login;	//用户名
	string                  m_psw;		//密码
	long					m_cnnFlg;   //连接成功标志
	string					m_dbguid;   //数据库GUID
	short					m_cnnType;	//服务类型
	Environment				*m_env;		//环境
	Connection				*m_conn;
	StatelessConnectionPool	*m_connPool;

	std::mutex				m_mutex;
	std::condition_variable m_condition;
	CriticalSection         cs;
};

CORLServer* pServer = new CORLServer;
Connection	* AcquireConn()
{
try
	{
		std::lock_guard<std::mutex> lk(pServer->m_mutex);
		if (NULL == pServer->m_connPool)
		{
			pServer->m_env = Environment::createEnvironment(Environment::THREADED_MUTEXED);

			//由于连接不上时连接池也会创建成功，故需要单独进行一次连接测试。
			pServer->m_conn = pServer->m_env->createConnection(szLog, szPsw, szSvr);
			if (pServer->m_conn != NULL)
			{
				pServer->m_env->terminateConnection(pServer->m_conn);
				pServer->m_conn = NULL;

				memset(&optionInfo, 0, sizeof(CLN_CFG_OPT));
				if (cfg_GetOptionInfo(CFG_NDXSERVER_CONNECTPOOL_SIZE, optionInfo))
				{
					lPoolSize = optionInfo.value.lngValue;
				}
				if (lPoolSize <= 0)
				{
					lPoolSize = SERVER_CONNECT_DEFAULT_MAX_POOLSIZE;
				}
				//创建连接池
				pServer->m_connPool = pServer->m_env->createStatelessConnectionPool(
					szLog, szPsw, szSvr,
					lPoolSize,
					SERVER_CONNECT_DEFAULT_MIN_POOLSIZE,
					SERVER_CONNECT_INCREAMENT_POOLSIZE);
				
				rtn = 1;
			}
		}

		std::unique_lock<std::mutex> lk(pServer->m_mutex);

		Connection *pConn = NULL;
		pConn = pServer->m_connPool->getConnection(pServer->m_Svr);
		while (pConn == NULL)
		{
		   pServer->m_condition.wait(lk);
		   pConn = pServer->m_connPool->getConnection(pServer->m_Svr);
		}
		return pConn;
	}
	catch (SQLException &sqlExcp)
	{
		int i = sqlExcp.getErrorCode();
		string strinfo = sqlExcp.getMessage();
        Disconnect();
	}
}

```
```
void Disconnect()
{
	try
	{
         cs.lock();
		if (pServer->m_env != NULL)
		{
			if (pServer->m_connPool != NULL)
			{
				pServer->m_env->terminateStatelessConnectionPool(pServer->m_connPool);
				pServer->m_connPool = NULL;
			}

			Environment::terminateEnvironment(pServer->m_env);
			pServer->m_env = NULL;
			pServer->m_cnnFlg = 0;
		}
		cs.UnLock();
	}
	catch (SQLException &sqlExcp)
	{
		int i = sqlExcp.getErrorCode();
		string strinfo = sqlExcp.getMessage();
       
	}
}
```	

# 171 Oracle search batch

```
long GetMaxShpLen()
{
	if (m_maxGemBlob != -1)
		return 1;
	string sql = "SELECT max(length(a.shape.points))  FROM ";
	sql += name;
	sql += " a ";
	Statement *stmt = NULL;
	ResultSet *rs = NULL;
	Connection *pConn = NULL;
	try
	{
		pConn = AcquireConn();
		stmt = pConn->createStatement();
		rs = stmt->executeQuery(sql);
		if (rs->next())
		{
			m_maxGemBlob = rs->getInt(1);
		}
		rtn = 1;
	}
	catch (SQLException &sqlExcp)
	{
		int i = sqlExcp.getErrorCode();
		string strinfo = sqlExcp.getMessage(); 
		error(strinfo, sql, i);
		rtn = 0;
	}

	if (stmt != NULL)
	{
		if (rs != NULL)
		{
			stmt->closeResultSet(rs);
			rs = NULL;
		}
		pConn->terminateStatement(stmt);
		stmt = NULL;
	}
	if (pConn != NULL)
	{
		ReleaseConn(pConn);
		pConn = NULL;
	}
	return rtn;
}

```

```
void SearchORLBatch()
{
	
	char					*ptOIDGem;			//要素ID缓冲区(字符串类型)
	int						*ptCbLenOIDGem;		//存OID字段NULL标志
	char					*ptShape;			    //要素shape缓冲区(BLOB类型)
	int						*ptShapePntNum;	    //点个数
	int					    *ptCbLenShape;	//存shape字段NULL标志
	int					    *ptCbLenShapePntNum;
    const int                m_batNum = 100;
	long                    m_maxGemBlob;

	
	Statement				*m_stmtGem;
	ResultSet				*m_rsGem;
	Connection				*m_connGem;
	Connection				*m_connAtt;
	string strSQGem = " SELECT OBJECTID,  a.SHAPE.points  ,  a.SHAPE.numpts  FROM SDE.road a  WHERE a.SHAPE.points IS NOT NULL  order by OBJECTID asc";
	try
	{
		m_connGem = AcquireConn();
		m_stmtGem = m_connGem->createStatement();
		if (m_isBatGem == false)
		{
			long  num = 0;
			GetObjNum(&num);
			m_stmtGem->setPrefetchRowCount(rowCount);
			m_stmtGem->setPrefetchMemorySize(bytes);
		}
		GetMaxShpLen();
		m_rsGem = m_stmtGem->executeQuery(strSQGem);

       {
		ptOIDGem = new char[m_batNum * 21];
		ptCbLenOIDGem = new int[m_batNum];
		memset(ptOIDGem, 0, m_batNum * 21);
		memset(ptCbLenOIDGem, 0, m_batNum * sizeof(int));

		ptShape = new char[m_batNum * m_maxGemBlob];

		memset(ptShape, 0, m_batNum * m_maxGemBlob);
		ptShapePntNum = new int[m_batNum];
		memset(ptShapePntNum, 0, m_batNum * sizeof(int));

		ptShape = new char[m_batNum * m_maxGemBlob];
		ptCbLenShape = new int[m_batNum];
		memset(ptCbLenShape, 0, m_batNum * sizeof(int));
		ptCbLenShapePntNum = new int[m_batNum];
		memset(ptCbLenShapePntNum, 0, m_batNum * sizeof(int));
    
       }
		m_rsGem->setDataBuffer(1, ptOIDGem, OCCI_SQLT_CHR, 21, (ub4*)ptCbLenOIDGem, NULL, NULL);
		m_rsGem->setDataBuffer(2, ptShape, OCCI_SQLT_LBI, m_maxGemBlob, (ub4*)ptCbLenShape, NULL, NULL);
		m_rsGem->setDataBuffer(3, ptShapePntNum, OCCIINT, sizeof(int), (ub4*)ptCbLenShapePntNum, NULL, NULL);

		////
		if (m_isBatGem)
		{
			try
			{
				stus = m_rsGem->next(m_batNum);
				fetchCnt = m_rsGem->getNumArrayRows();
				rtn = 1;
			}
			catch (SQLException &sqlExcp)
			{
				int i = sqlExcp.getErrorCode();
				string strinfo = sqlExcp.getMessage(); 
			}
		}
		else{
		ResultSet::Status stus = m_rsGem->next();
		if (stus == ResultSet::END_OF_FETCH)
		{
			return(GET_NO_DATA);
		}
		oid = m_rsGem->getInt(1);
		Blob shapeBlob = m_rsGem->getBlob(2);
		int pntNum = 0;
		pntNum = m_rsGem->getInt(3);
		bool isNULL = shapeBlob.isNull();
		if (isNULL == false)
		{
			int lenBlob = shapeBlob.length();
			char *chr_out = new char[lenBlob];
			memset(chr_out, 0, lenBlob);
			shapeBlob.open(OCCI_LOB_READONLY);
			shapeBlob.read(lenBlob, (unsigned char*)chr_out, lenBlob, 1);
			shapeBlob.close();
		}

	}
		
	int m_curFetchPos = 0;
	for (int i=0; i<fetchCnt; i++)
	{
		int nOid = _atoi64(ptOIDGem + 21 * m_curFetchPos);
		char *ptWKB = ptShape + m_maxGemBlob * m_curFetchPos;
		int lenBlob = ptCbLenShape[m_curFetchPos];
		int pntNum = 0;
		pntNum = ptShapePntNum[m_curFetchPos];
	}

	}
	catch (SQLException &sqlExcp)
	{
		int i = sqlExcp.getErrorCode();
		string strinfo = sqlExcp.getMessage(); 
	}
	
	if (m_stmtGem != NULL)
	{
		if (m_rsGem != NULL)
		{
			m_stmtGem->closeResultSet(m_rsGem);
			m_rsGem = NULL;
		}
		m_connGem->terminateStatement(m_stmtGem);
		m_stmtGem = NULL;
	}

}

```	


# 172 view OSM road

```
1. download osm file 'china-latest.osm.pbf' from  http://download.geofabrik.de/asia.html

2. import osm data into postgreSQL with osm2pgsql with following command
osm2pgsql -d osm_china -U postgres -P 5432 -C 12000 -S "F:/osm2pgsql-bin/default.style" F:/osmdata/china-latest.osm.pbf

3. open QGIS and then link postgresql database 'osm_china' ;

4. if link ok, gather the layer 'planet_osm_roads' into view window in QGIS;

5.open the corresponding property window, symbolize the roads with some color and style ,set the matched annotations,then we can see the interested region in the view;

```

# 173 open close  read write file

```
#define MAX_LEN 512
typedef  struct 
{
	//
	char	fileType[16];	//文件验证码：CYCLECFGFILE
	short	version;     	//配置文件的版本号()

    //1. 系统ID记录
	long	maxConnID;			//目前系统中分配的最大连接ID
	long	maxGdbID;			//目前系统中分配的最大GDB ID
	long	maxUserID;			//目前系统中分配的最大用户ID

	//2.连接信息表
	long	connOff;				//连接信息表起始位置
	long	connNum;				//连接的数目，即有效的连接数，也是连接信息表的记录数。
	long	connBufLen;			    //预留的连接信息表的记录数目，即为连接表预留记录数。
	
	//4.用户信息表;	
	long	userOff;             //用户信息表的起始地址
	long	userNum;             //用户信息数目,即有效的用户数,也是用户信息表的记录数
	long	userBufLen;          //预留的用户信息表的记录数目,即为用户信息表预留的记录数
	
	//5.用户权限信息表
	long	privOff;			//用户权限信息表的地址;
	long	privNum;			//用户权限信息表的记录数
	long    privBufLen;         //预留的用户权限表的记录数目,即为用户权限表预留的记录数

	//7.临时目录路径
	char  tempDirName[MAX_LEN];
	//8. 保留
    char  res[512];   
}GLOBAL_INFO;	

typedef struct USER_INFO
{
	long	userID;		
	char	userName[MAX_LEN];
	char    pswd[MAX_LEN];	
}USER_INFO;


HANDLE OpenFile()
{
	HANDLE hFile=NULL;
	hFile=CreateFile(
					g_CfgFName,
					GENERIC_READ|GENERIC_WRITE,
					FILE_SHARE_READ|FILE_SHARE_WRITE,
					NULL,
					OPEN_EXISTING,
					NULL,
					NULL);	
	return (hFile);
}

long  CloseFile(HANDLE hFile)
{
	if(hFile!=INVALID_HANDLE_VALUE)
		CloseHandle(hFile);
	return(1);
}

long	SetGlobalInfo(HANDLE hFile,GLOBAL_INFO & cfgInfo)
{
	DWORD	off=0; 
	
	SetFilePointer(hFile,0,NULL,FILE_BEGIN);
	WriteFile(hFile,&cfgInfo,sizeof(GLOBAL_INFO),&off,NULL);
	return (1);
}

long	GetGlobalInfo(HANDLE hFile,GLOBAL_INFO & cfgInfo)
{
	DWORD	off=0; 
	
	SetFilePointer(hFile,0,NULL,FILE_BEGIN);
	ReadFile(hFile,&cfgInfo,sizeof(GLOBAL_INFO),&off,NULL);
	return (1);
}

long 	UpdateUser(long userID,char * pswd)
{
	HANDLE			hFile;
	GLOBAL_INFO	    cfgInfo;
	USER_INFO       userInfo;
	DWORD           off=0;
	long            rtn=0;
	long			woff=0;
	
	if(userID <=0)
		return (0);
	if(pswd ==NULL)
		return (0);
	
	//1.打开文件
	hFile =OpenFile();
	if(hFile == INVALID_HANDLE_VALUE)
		return (0);
	
	//2.取文件全局信息
	GetGlobalInfo(hFile,cfgInfo);	
	
	//3.比较，取对应ID的记录的信息
	SetFilePointer(hFile,cfgInfo.userOff,NULL,FILE_BEGIN);
	for(long i=0;i<cfgInfo.userNum;i++)
	{
		ReadFile(hFile,&userInfo,sizeof(USER_INFO),&off,NULL);	
		if(userID ==userInfo.userID)
		{			
		strncpy(userInfo.pswd,pswd,MAX_LEN);
		woff=sizeof(USER_INFO) * i;
		SetFilePointer(hFile,cfgInfo.userOff+woff,NULL,FILE_BEGIN);
		WriteFile(hFile,&userInfo,sizeof(USER_INFO),&off,NULL);
		rtn=1;
		break;
		}
	}
	//4.关闭文件
	CloseFile(hFile);
	return (rtn);	
}

long 	GetUserIDs(std::vector<long> &IDs)
{
	HANDLE				hFile;
	GLOBAL_INFO		    cfgInfo;
	USER_INFO	    	info;
	DWORD               off=0;
	long                rtn=0;
	
	//0.清空输出信息
	IDs.clear();
	
	//1.打开文件
	hFile =OpenFile();
	if(hFile== INVALID_HANDLE_VALUE)
		return (0);
				
	//2.取文件全局信息
	GetGlobalInfo(hFile,cfgInfo);				
				
	//4.取连接信息
	SetFilePointer(hFile,cfgInfo.userOff,NULL,FILE_BEGIN);
	for(long i=0;i<cfgInfo.userNum;i++)
		{
		ReadFile(hFile,&info,sizeof(USER_INFO),&off,NULL);	
		rtn=IDs.pushback(info.userID);
		}
	
	//5.关闭文件
	CloseFile(hFile);

	return (IDs.Size());		
}




```

# 174 get disk information type  space


```

#include "direct.h"
#include "io.h"

typedef struct  tag_DriveInfo
{
	ULONGLONG  freeSpace;   //可用空间(字节为单位)
	ULONGLONG  totalSpace;  //总空间(字节为单位)
	char       diskType;    //磁盘类型（windows.h中定义的磁盘类型）：
							//#define DRIVE_UNKNOWN     0
							//#define DRIVE_NO_ROOT_DIR 1 
							//#define DRIVE_REMOVABLE   2
							//#define DRIVE_FIXED       3
							//#define DRIVE_REMOTE      4
							//#define DRIVE_CDROM       5
							//#define DRIVE_RAMDISK     6
	char    diskSign[4];    //盘符
}DRIVER_INFO; 

//功能：取当前计算机的磁盘数
//参数：无
//返回：成功返回磁盘数（>=0），失败返回-1
long  GetDiskNum() 
{
	int		num = 0;
	char    strVol[16];
	char    i;
	DWORD   msk;
	DWORD   drvMsk;
	UINT    drvType;

	msk=0x01;
	drvMsk=GetLogicalDrives();

	strcpy(strVol,"A:");
	for(i=0;i<26;i++)
	{
		if((drvMsk & msk))
			{
			drvType=GetDriveType(strVol);
			if((drvType == DRIVE_FIXED) || (drvType==DRIVE_REMOTE))  
				num++;
			}
		msk=msk<<1;
		strVol[0]++;
	}
	return(num);
}


//功能：取给定盘符磁盘的可用空间和总空间
//参数：diskSign[in]         -  传入char类型的磁盘盘符（如C盘为'C')。
//      freeSpace[in, out]   -  传入ULONGLONG类型参数的地址，返回磁盘可用空间大小（字节为单位）
//      totalSpace[in, out]  -  传入ULONGLONG类型参数的地址，返回磁盘总空间大小（字节为单位）
//返回：成功返回1，失败返回-1
long  GetDiskSpace(char diskSign,ULONGLONG *freeSpace, ULONGLONG *totalSpace)
{
		char	        strVol[3];
		UINT			drvType;
		_ULARGE_INTEGER result_freeSpace;
		_ULARGE_INTEGER result_totalSpace;
		_ULARGE_INTEGER result_g;
	
	//strVol.Format("%c:",diskSign);
	strVol[0]=diskSign;
	strVol[1]=':';
	strVol[2]=0;

	drvType = (UINT)GetDriveType(strVol);
	
	if(drvType != DRIVE_FIXED)  //是否是硬盘
	if((drvType != DRIVE_FIXED) && 
	   (drvType != DRIVE_REMOTE))  
		return -1;

	GetDiskFreeSpaceEx(strVol, &result_freeSpace, &result_totalSpace, &result_g);	
	*freeSpace = result_freeSpace.QuadPart;
	*totalSpace= result_totalSpace.QuadPart;
	return(1);
}


//功能：获取给定磁盘信息（磁盘盘符，磁盘类型，总空间，可用空间）
//参数：dskSign[in]       - 传入char类型的磁盘盘符
//      driveInfo[in out]   - 传入DRIVER_INFO结构体参数地址，返回相应的磁盘信息
//返回：成功返回1，失败返回-1
long  GetDiskInfo(char dskSign, DRIVER_INFO* driveInfo)
{
	char    strVol[3];
	int        diskType;
	_ULARGE_INTEGER result_freeSpace;
	_ULARGE_INTEGER result_totalSpace;
	_ULARGE_INTEGER result_g;
	
	strVol[0]=dskSign;
	strVol[1]=':';
	strVol[2]=0;

	if(GetDiskType(dskSign, &diskType) >= 0)
	{
	strcpy(driveInfo->diskSign,strVol);
	driveInfo->diskType = diskType;
	if(GetDiskFreeSpaceEx(strVol, &result_freeSpace, &result_totalSpace, &result_g)!=-1)
		{
		driveInfo->freeSpace = result_freeSpace.QuadPart;
		driveInfo->totalSpace= result_totalSpace.QuadPart;
		return(1);
		}
	}
	return(-1);
}


//功能：取给定盘符磁盘的类型
//参数：diskSign[in]     -  传入char类型的磁盘盘符
//      diskType[in out] -  传入int型参数地址，返回磁盘类型，
//返回：成功返回1，失败返回-1
long  GetDiskType(char diskSign,int *diskType)
{
	char strVol[3];

	strVol[0]=diskSign;
	strVol[1]=':';
	strVol[2]=0;

	*diskType = GetDriveType(strVol);
	
	if((*diskType != DRIVE_UNKNOWN) && (*diskType != DRIVE_NO_ROOT_DIR))
		return 1;
	else
		return -1;
}

//功能：取当前计算机的所有磁盘盘符
//参数：ptDiskSign[in,out] - 传入char类型数组地址，返回磁盘盘符（如C盘为'C')。
//      len       [in]     - 传入ptDiskSign的长度
//返回：成功返回磁盘数(>=0)，失败返回-1
long  GetDiskSign(char *ptDiskSign, long len)
{
		long	diskNo = 0;
		char    i;
		char    strVol[16];
		DWORD   msk;
		DWORD   drvMsk;
		UINT    drvType;

	msk=0x01;
	drvMsk=GetLogicalDrives();

	strcpy(strVol,"A:");
	for(i=0;i<26;i++)
	{
		if((drvMsk & msk))
		{
			drvType=GetDriveType(strVol);
			if((drvType == DRIVE_FIXED) || (drvType==DRIVE_REMOTE))  
			{
			ptDiskSign[diskNo] = strVol[0];
			diskNo++;
			if(diskNo>=len)
				break;
			}
		}
		msk=msk<<1;
		strVol[0]++;
	}
	return(diskNo);
}

```


# 175 CREATE TABLE public.planet_osm_roads  planet_osm_polygon

## 1. planet_osm_roads

```
-- Table: public.planet_osm_roads

-- DROP TABLE public.planet_osm_roads;

CREATE TABLE public.planet_osm_roads
(
    osm_id bigint,
    access text COLLATE pg_catalog."default",
    "addr:housename" text COLLATE pg_catalog."default",
    "addr:housenumber" text COLLATE pg_catalog."default",
    "addr:interpolation" text COLLATE pg_catalog."default",
    admin_level text COLLATE pg_catalog."default",
    aerialway text COLLATE pg_catalog."default",
    aeroway text COLLATE pg_catalog."default",
    amenity text COLLATE pg_catalog."default",
    barrier text COLLATE pg_catalog."default",
    bicycle text COLLATE pg_catalog."default",
    bridge text COLLATE pg_catalog."default",
    boundary text COLLATE pg_catalog."default",
    building text COLLATE pg_catalog."default",
    construction text COLLATE pg_catalog."default",
    covered text COLLATE pg_catalog."default",
    foot text COLLATE pg_catalog."default",
    highway text COLLATE pg_catalog."default",
    historic text COLLATE pg_catalog."default",
    horse text COLLATE pg_catalog."default",
    junction text COLLATE pg_catalog."default",
    landuse text COLLATE pg_catalog."default",
    layer integer,
    leisure text COLLATE pg_catalog."default",
    lock text COLLATE pg_catalog."default",
    man_made text COLLATE pg_catalog."default",
    military text COLLATE pg_catalog."default",
    name text COLLATE pg_catalog."default",
    "natural" text COLLATE pg_catalog."default",
    oneway text COLLATE pg_catalog."default",
    place text COLLATE pg_catalog."default",
    power text COLLATE pg_catalog."default",
    railway text COLLATE pg_catalog."default",
    ref text COLLATE pg_catalog."default",
    religion text COLLATE pg_catalog."default",
    route text COLLATE pg_catalog."default",
    service text COLLATE pg_catalog."default",
    shop text COLLATE pg_catalog."default",
    surface text COLLATE pg_catalog."default",
    tourism text COLLATE pg_catalog."default",
    tracktype text COLLATE pg_catalog."default",
    tunnel text COLLATE pg_catalog."default",
    water text COLLATE pg_catalog."default",
    waterway text COLLATE pg_catalog."default",
    way_area real,
    z_order integer,
    way geometry(LineString,3857)
)

TABLESPACE pg_default;

ALTER TABLE public.planet_osm_roads
    OWNER to postgres;

-- Index: planet_osm_roads_way_idx

-- DROP INDEX public.planet_osm_roads_way_idx;

CREATE INDEX planet_osm_roads_way_idx
    ON public.planet_osm_roads USING gist
    (way)
    WITH (FILLFACTOR=100)
    TABLESPACE pg_default;

```

## 2. planet_osm_polygon

```
-- Table: public.planet_osm_polygon

-- DROP TABLE public.planet_osm_polygon;

CREATE TABLE public.planet_osm_polygon
(
    osm_id bigint,
    access text COLLATE pg_catalog."default",
    "addr:housename" text COLLATE pg_catalog."default",
    "addr:housenumber" text COLLATE pg_catalog."default",
    "addr:interpolation" text COLLATE pg_catalog."default",
    admin_level text COLLATE pg_catalog."default",
    aerialway text COLLATE pg_catalog."default",
    aeroway text COLLATE pg_catalog."default",
    amenity text COLLATE pg_catalog."default",
    barrier text COLLATE pg_catalog."default",
    bicycle text COLLATE pg_catalog."default",
    bridge text COLLATE pg_catalog."default",
    boundary text COLLATE pg_catalog."default",
    building text COLLATE pg_catalog."default",
    construction text COLLATE pg_catalog."default",
    covered text COLLATE pg_catalog."default",
    foot text COLLATE pg_catalog."default",
    highway text COLLATE pg_catalog."default",
    historic text COLLATE pg_catalog."default",
    horse text COLLATE pg_catalog."default",
    junction text COLLATE pg_catalog."default",
    landuse text COLLATE pg_catalog."default",
    layer integer,
    leisure text COLLATE pg_catalog."default",
    lock text COLLATE pg_catalog."default",
    man_made text COLLATE pg_catalog."default",
    military text COLLATE pg_catalog."default",
    name text COLLATE pg_catalog."default",
    "natural" text COLLATE pg_catalog."default",
    oneway text COLLATE pg_catalog."default",
    place text COLLATE pg_catalog."default",
    power text COLLATE pg_catalog."default",
    railway text COLLATE pg_catalog."default",
    ref text COLLATE pg_catalog."default",
    religion text COLLATE pg_catalog."default",
    route text COLLATE pg_catalog."default",
    service text COLLATE pg_catalog."default",
    shop text COLLATE pg_catalog."default",
    surface text COLLATE pg_catalog."default",
    tourism text COLLATE pg_catalog."default",
    tracktype text COLLATE pg_catalog."default",
    tunnel text COLLATE pg_catalog."default",
    water text COLLATE pg_catalog."default",
    waterway text COLLATE pg_catalog."default",
    way_area real,
    z_order integer,
    way geometry(Geometry,3857)
)

TABLESPACE pg_default;

ALTER TABLE public.planet_osm_polygon
    OWNER to postgres;

-- Index: planet_osm_polygon_way_idx

-- DROP INDEX public.planet_osm_polygon_way_idx;

CREATE INDEX planet_osm_polygon_way_idx
    ON public.planet_osm_polygon USING gist
    (way)
    WITH (FILLFACTOR=100)
    TABLESPACE pg_default;
```
## 3. planet_osm_point

```
-- Table: public.planet_osm_point

-- DROP TABLE public.planet_osm_point;

CREATE TABLE public.planet_osm_point
(
    osm_id bigint,
    access text COLLATE pg_catalog."default",
    "addr:housename" text COLLATE pg_catalog."default",
    "addr:housenumber" text COLLATE pg_catalog."default",
    admin_level text COLLATE pg_catalog."default",
    aerialway text COLLATE pg_catalog."default",
    aeroway text COLLATE pg_catalog."default",
    amenity text COLLATE pg_catalog."default",
    barrier text COLLATE pg_catalog."default",
    boundary text COLLATE pg_catalog."default",
    building text COLLATE pg_catalog."default",
    highway text COLLATE pg_catalog."default",
    historic text COLLATE pg_catalog."default",
    junction text COLLATE pg_catalog."default",
    landuse text COLLATE pg_catalog."default",
    layer integer,
    leisure text COLLATE pg_catalog."default",
    lock text COLLATE pg_catalog."default",
    man_made text COLLATE pg_catalog."default",
    military text COLLATE pg_catalog."default",
    name text COLLATE pg_catalog."default",
    "natural" text COLLATE pg_catalog."default",
    oneway text COLLATE pg_catalog."default",
    place text COLLATE pg_catalog."default",
    power text COLLATE pg_catalog."default",
    railway text COLLATE pg_catalog."default",
    ref text COLLATE pg_catalog."default",
    religion text COLLATE pg_catalog."default",
    shop text COLLATE pg_catalog."default",
    tourism text COLLATE pg_catalog."default",
    water text COLLATE pg_catalog."default",
    waterway text COLLATE pg_catalog."default",
    way geometry(Point,3857)
)

TABLESPACE pg_default;

ALTER TABLE public.planet_osm_point
    OWNER to postgres;

-- Index: planet_osm_point_way_idx

-- DROP INDEX public.planet_osm_point_way_idx;

CREATE INDEX planet_osm_point_way_idx
    ON public.planet_osm_point USING gist
    (way)
    WITH (FILLFACTOR=100)
    TABLESPACE pg_default;
```
## 4. planet_osm_line

```
-- Table: public.planet_osm_line

-- DROP TABLE public.planet_osm_line;

CREATE TABLE public.planet_osm_line
(
    osm_id bigint,
    access text COLLATE pg_catalog."default",
    "addr:housename" text COLLATE pg_catalog."default",
    "addr:housenumber" text COLLATE pg_catalog."default",
    "addr:interpolation" text COLLATE pg_catalog."default",
    admin_level text COLLATE pg_catalog."default",
    aerialway text COLLATE pg_catalog."default",
    aeroway text COLLATE pg_catalog."default",
    amenity text COLLATE pg_catalog."default",
    barrier text COLLATE pg_catalog."default",
    bicycle text COLLATE pg_catalog."default",
    bridge text COLLATE pg_catalog."default",
    boundary text COLLATE pg_catalog."default",
    building text COLLATE pg_catalog."default",
    construction text COLLATE pg_catalog."default",
    covered text COLLATE pg_catalog."default",
    foot text COLLATE pg_catalog."default",
    highway text COLLATE pg_catalog."default",
    historic text COLLATE pg_catalog."default",
    horse text COLLATE pg_catalog."default",
    junction text COLLATE pg_catalog."default",
    landuse text COLLATE pg_catalog."default",
    layer integer,
    leisure text COLLATE pg_catalog."default",
    lock text COLLATE pg_catalog."default",
    man_made text COLLATE pg_catalog."default",
    military text COLLATE pg_catalog."default",
    name text COLLATE pg_catalog."default",
    "natural" text COLLATE pg_catalog."default",
    oneway text COLLATE pg_catalog."default",
    place text COLLATE pg_catalog."default",
    power text COLLATE pg_catalog."default",
    railway text COLLATE pg_catalog."default",
    ref text COLLATE pg_catalog."default",
    religion text COLLATE pg_catalog."default",
    route text COLLATE pg_catalog."default",
    service text COLLATE pg_catalog."default",
    shop text COLLATE pg_catalog."default",
    surface text COLLATE pg_catalog."default",
    tourism text COLLATE pg_catalog."default",
    tracktype text COLLATE pg_catalog."default",
    tunnel text COLLATE pg_catalog."default",
    water text COLLATE pg_catalog."default",
    waterway text COLLATE pg_catalog."default",
    way_area real,
    z_order integer,
    way geometry(LineString,3857)
)

TABLESPACE pg_default;

ALTER TABLE public.planet_osm_line
    OWNER to postgres;

-- Index: planet_osm_line_way_idx

-- DROP INDEX public.planet_osm_line_way_idx;

CREATE INDEX planet_osm_line_way_idx
    ON public.planet_osm_line USING gist
    (way)
    WITH (FILLFACTOR=100)
    TABLESPACE pg_default;
```

# 176  get oracle version


## 1. 在PL/SQL中执行下面语句，即可获取当前oracle版本号：
```
Select product, version,status FROM Product_component_version     Where SUBSTR(PRODUCT,0,6)='Oracle';


Oracle Database 12c Enterprise Edition 	12.2.0.1.0	64bit Production

```

## 2. 使用navicat查看
首先需要下载安装navicat然后连接Oracle数据库；

然后执行查询Oracle版本的命令，效果如下图所示

select * from v$version


## 3. 查看安装的Oracle客户端版本

如下所示，使用sqlplus -v命令，可以查到该客户端安装的 11.2.0.1.0的客户端版本。

C:\Users>sqlplus -v

SQL*Plus: Release 11.2.0.1.0 Production


# 177  create and debug dump file in linux

## create core dump

```
for core-dump file in linux
1. check if Failed to write core dump, use 'ulimit -c '
if we get the return value '0' which means we cannot write core dump;
if wanna write core dump,use command:

ulimit -c unlimited

2. set dump location

[root@king ~]# mkdir /dump/
[root@king ~]# echo '/dump/core-%e-%p-%t' > /proc/sys/kernel/core_pattern
[root@king ~]# more /proc/sys/kernel/core_pattern 
/dump/core-%e-%p-%t

```
## debug dump

```

3. run target execute,if crash,get the dumpfile in folder '/dump/' 

4. debug dump file 

[root@king ~]# gdb core-file core-igserverjava-13430-1698032409
[root@king ~]# set solib-search-path /data/kingserver/java/program:/lib/aarch64-linux-gnu:/data//kingserver/java/program/java/jre/lib/aarch64:/data/kingserver/java/program/java/jre/lib/aarch64/jli:/data/kingserver/java/program/QtPlugins/imageformats
[root@king ~]# info sharedlibrary
[root@king ~]# bt
[root@king ~]# thread apply all bt

```

```
gdb core-file core-serverjava-13430-1698032409

set solib-search-path /home/zone/program/:/lib/aarch64-linux-gnu:/home/zone/program/java/jre/lib/aarch64:/home/zone/program/java/jre/lib/aarch64/jli:/home/zone/program/QtPlugins/imageformats

info sharedlibrary

thread apply all bt
```


# 178　debug multiple thread process with gdb  in linux

```
1. root@king80105:/home/king#  ps -aux | grep kingserverjava
root       29255  8.5  4.1 8038568 671984 pts/2  Sl   19:41   0:19 /program/java/jre/bin/kingserverjava -Ddcs.dubboPort=50080 -Dcycle.home=.. -Digs.home=. -Dloader.path=../program/java/lib/hdfs -jar /kingserver/java/lib/dcserver-host-10.6.6.10.jar
root       29308 45.8 11.9 10222152 1955008 pts/2 Sl  19:41   1:43 /program/java/jre/bin/kingserverjava -Dfile.encoding=UTF-8 -Dcycle.home=.. -Digs.home=. -jar /kingserver/java/lib/igserver-webapp-10.6.6.10.jar
root       31140  0.0  0.0  11656   668 pts/2    S+   19:45   0:00 grep --color=auto kingserverjava


2. root@king80105:/home/king# netstat -tnlp
激活Internet连接 (仅服务器)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:5900            0.0.0.0:*               LISTEN      2167/vino-server
tcp        0      0 127.0.0.1:8752          0.0.0.0:*               LISTEN      746/kysecdbd
tcp        0      0 0.0.0.0:54321           0.0.0.0:*               LISTEN      1370/kingbase
tcp        0      0 0.0.0.0:7250            0.0.0.0:*               LISTEN      2407/miracle-agent
tcp        0      0 127.0.0.1:53            0.0.0.0:*               LISTEN      1071/dnsmasq
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      749/systemd-resolve
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1105/sshd: /usr/sbi
tcp        0      0 127.0.0.1:6011          0.0.0.0:*               LISTEN      26129/sshd: root@pt
tcp6       0      0 :::4236                 :::*                    LISTEN      1092/dmap
tcp6       0      0 :::80                   :::*                    LISTEN      1161/nginx: master
tcp6       0      0 :::54321                :::*                    LISTEN      1370/kingbase

3. locate the target port 54321,get the matched PID '1370'
   then debug the process with target pid 

4. root@king80105:/home/king# gdb --pid=1370
int gdb command env, set the lib path with 'set  solib-search-path '; such as  
set  solib-search-path /home/kingserver/program

if encounter SIGSEGV， gdb will stop, then ignore it with following command:

handle SIGSEGV nostop noprint
```

# 179.  CMemoryMnger

```

//内存分配管理类
class  CMemoryMnger                           
{
public:
	 CMemoryMnger(void);
	~CMemoryMnger(void);

public:
	void *Alloc(DWORD size);                //分配空间
	long long Free();						//重新回到空间开始位置
	long long Reset();                       //重新回到空间开始位置

private:
	std::vector<char *> m_BufferSet;
	char			   *m_pBuff;
	long long			m_idx;
	DWORD				m_pos;
};

```

```
#define MEM_BLOCK_SIZE	(1024*1024*10)	//10M 空间

CMemoryMnger::CMemoryMnger(void)
{
	m_pBuff=NULL;
	m_pos=0;              //记录每一块的位置
	m_idx=0;              //记录块序号
}

CMemoryMnger::~CMemoryMnger(void)
{
	for (long long i=0;i<m_BufferSet.Size();i++)
	{
		delete []m_BufferSet[i];
	}
}

void *CMemoryMnger::Alloc(DWORD size)
{
	if(m_pBuff==NULL)
	{
		m_pBuff=new char[MEM_BLOCK_SIZE];
		m_BufferSet.push_back(m_pBuff);
	}
	//m_pBuff
	if(MEM_BLOCK_SIZE<size+m_pos)
	{
		m_idx++;
		if(m_idx>=m_BufferSet.Size())
		{
			m_pBuff=new char[((MEM_BLOCK_SIZE>size)?MEM_BLOCK_SIZE:size)];
			m_BufferSet.push_back(m_pBuff);
		}
		else if(MEM_BLOCK_SIZE>=size)
			m_pBuff=m_BufferSet[m_idx];
		else
		{
			delete []m_BufferSet[m_idx];
			m_pBuff=new char[size];
			m_BufferSet.insert(m_idx,m_pBuff);
		}
		m_pos=0;
	}
	
	//分配
	void *rtn=m_pBuff+m_pos;
	m_pos+=size;
	return(rtn);
}

long long CMemoryMnger::Free()       
{
	if(m_pBuff==NULL)return(1);

	m_pBuff=m_BufferSet[0];
	m_pos=0;
	m_idx=0;
	return(1);
}

long long CMemoryMnger::Reset()
{
	return(Free());
}


```


# 180  CPhysicalMemory

```

//物理内存管理器，提供比new类函数更高效，更灵活，更底层的内存管理服务
namespace CPhysicalMemory
{
	//long long  Init();//初始化
	void __EXPORT_API_ *MemoryAlloc(DWORD size);
	void __EXPORT_API_ *VirtualAlloc(DWORD size);
	void __EXPORT_API_ *PhysicalAlloc(DWORD size);
	BOOL __EXPORT_API_ MapMemory(void *virMem,void *phyMem);
	BOOL __EXPORT_API_ UnMapMemory(void *virMem,void *phyMem);
	BOOL __EXPORT_API_ VirtualCopy(void *virDst,void *virSrc,void *phyMem);
	BOOL __EXPORT_API_ VirtualSwap(void *virDst,void *phyDst,void *virSrc,void *phySrc);
	BOOL __EXPORT_API_ VirtualFree(void *virMem);
	BOOL __EXPORT_API_ PhysicalFree(void *phyMem);
	BOOL __EXPORT_API_ GetStatus();
};
```

```

BOOL LoggedSetLockPagesPrivilege ( HANDLE hProcess,
                              BOOL bEnable);

struct tagPhyMem
{
	ULONG_PTR NumberOfPages;        // number of pages to request
	ULONG_PTR *aPFNs;				// page info; holds opaque data
};

BOOL		   g_bStatus;
long long	   g_dwPageSize;

long long PhyMemoryInit()
{
	//return(FALSE);
	SYSTEM_INFO sSysInfo;           // useful system information
	GetSystemInfo(&sSysInfo);  // fill the system information structure
	g_dwPageSize=sSysInfo.dwPageSize;

	// Enable the privilege.

  if( ! LoggedSetLockPagesPrivilege( GetCurrentProcess(), TRUE ) ) 
  {
    return(0);
  }
  g_bStatus=TRUE;
  return(1);
}

void *CPhysicalMemory::MemoryAlloc(DWORD size)
{
	return ::VirtualAlloc( NULL,
                           size,
                           MEM_COMMIT,
                           PAGE_READWRITE );
}

void *CPhysicalMemory::VirtualAlloc(DWORD size)
{
	return ::VirtualAlloc( NULL,
                           size,
                           MEM_RESERVE | MEM_PHYSICAL,
                           PAGE_READWRITE );
}

BOOL  CPhysicalMemory::GetStatus()
{
	return(g_bStatus);
}

void  loger(const char* pInfo)
{
#ifdef _WIN32
	FILE *fp=fopen("c:\\memerror.log","a+");
	fprintf(fp,pInfo);
	fclose(fp);
#endif
}

void *CPhysicalMemory::PhysicalAlloc(DWORD size)
{
	BOOL bResult;
	ULONG_PTR NumberOfPagesInitial; // initial number of pages requested
	tagPhyMem	*pt=new tagPhyMem;
	pt->NumberOfPages = (size+g_dwPageSize-1)/g_dwPageSize;
	int PFNArraySize = pt->NumberOfPages * sizeof (ULONG_PTR);
	pt->aPFNs = (ULONG_PTR *) HeapAlloc(GetProcessHeap(), 0, PFNArraySize);
	if(pt->aPFNs==NULL)
		goto END;

	NumberOfPagesInitial = pt->NumberOfPages;
    bResult = AllocateUserPhysicalPages( GetCurrentProcess(),
                                       &(pt->NumberOfPages),
                                       pt->aPFNs );
	if( bResult != TRUE )
		goto END1;
	if( NumberOfPagesInitial != pt->NumberOfPages )
		goto END2;
	return(pt);

END2:
	FreeUserPhysicalPages( GetCurrentProcess(),
                                   &(pt->NumberOfPages),
                                   pt->aPFNs );
	{
	loger("申请物理内存数量不够\n\n");
	}

END1:
	HeapFree(GetProcessHeap(), 0, pt->aPFNs);
	{
	loger("申请物理内存失败\n\n");
	}

END:
	if(pt)delete pt;
	{
	   loger("申请堆失败\n\n");
	}

	return(NULL);
}

BOOL  CPhysicalMemory::MapMemory(void *virMem,void *phyMem)
{
	return MapUserPhysicalPages( virMem,
                                  ((tagPhyMem*)phyMem)->NumberOfPages,
                                  ((tagPhyMem*)phyMem)->aPFNs );
}

BOOL  CPhysicalMemory::UnMapMemory(void *virMem,void *phyMem)
{
	return MapUserPhysicalPages( virMem,
                                  ((tagPhyMem*)phyMem)->NumberOfPages,
                                  NULL );
}

BOOL  CPhysicalMemory::VirtualCopy(void *virDst,void *virSrc,void *phyMem)
{
	MapUserPhysicalPages( virSrc,
                                 ((tagPhyMem*)phyMem)->NumberOfPages,
                                  NULL );
	return MapUserPhysicalPages( virDst,
                                  ((tagPhyMem*)phyMem)->NumberOfPages,
                                  ((tagPhyMem*)phyMem)->aPFNs );
}

BOOL  CPhysicalMemory::VirtualSwap(void *virDst,void *phyDst,void *virSrc,void *phySrc)
{
	return(FALSE);
}

BOOL CPhysicalMemory::VirtualFree(void *virMem)
{
	return ::VirtualFree( virMem,
                         0,
                         MEM_RELEASE );
}

BOOL CPhysicalMemory::PhysicalFree(void *phyMem)
{
	BOOL rtn=FreeUserPhysicalPages( GetCurrentProcess(),
                                   &(((tagPhyMem*)phyMem)->NumberOfPages),
                                   ((tagPhyMem*)phyMem)->aPFNs);
	HeapFree(GetProcessHeap(), 0, ((tagPhyMem*)phyMem)->aPFNs);
	delete (tagPhyMem*)phyMem;
	return(rtn);
}


/*****************************************************************
   LoggedSetLockPagesPrivilege: a function to obtain or
   release the privilege of locking physical pages.
   Inputs:
       HANDLE hProcess: Handle for the process for which the
       privilege is needed
       BOOL bEnable: Enable (TRUE) or disable?
   Return value: TRUE indicates success, FALSE failure.

*****************************************************************/

BOOL
LoggedSetLockPagesPrivilege ( HANDLE hProcess,
                              BOOL bEnable)
{
  struct {
    DWORD Count;
    LUID_AND_ATTRIBUTES Privilege [1];
  } Info;

  HANDLE Token;
  BOOL Result;

  // Open the token.

  Result = OpenProcessToken ( hProcess,
                              TOKEN_ADJUST_PRIVILEGES|TOKEN_QUERY,
                              & Token);

  if( Result != TRUE ) 
  {
    //printf( "Cannot open process token.\n" );
    return FALSE;
  }

  // Enable or disable?

  Info.Count = 1;
  if( bEnable ) 
  {
    Info.Privilege[0].Attributes = SE_PRIVILEGE_ENABLED;
  } 
  else 
  {
    Info.Privilege[0].Attributes = 0;
  }

  // Get the LUID.

  Result = LookupPrivilegeValue ( NULL,
                                  SE_LOCK_MEMORY_NAME,
                                  &(Info.Privilege[0].Luid));

  if( Result != TRUE ) 
  {
    //printf( "Cannot get privilege for %s.\n", SE_LOCK_MEMORY_NAME );
    return FALSE;
  }

  // Adjust the privilege.

  Result = AdjustTokenPrivileges ( Token, FALSE,
                                   (PTOKEN_PRIVILEGES) &Info,
                                   0, NULL, NULL);

  // Check the result.
  if( Result != TRUE ) 
  {
    //printf ("Cannot adjust token privileges (%u)\n", GetLastError() );
    return FALSE;
  } 
  else 
  {
	  DWORD e;
    if( (e=GetLastError()) != ERROR_SUCCESS ) 
    {
      //printf ("Cannot enable the SE_LOCK_MEMORY_NAME privilege; ");
      //printf ("please check the local policy.\n");
      return FALSE;
    }
  }

  CloseHandle( Token );

  return TRUE;
}

```

# 181 

## get current free space
通过statvfs得到磁盘的空闲块数量(f_bfree)，以及每个空闲块大小(f_bsize)。

```
#include <cstdio>
#include <sys/statvfs.h>
#include <cerrno>
#include <cstring>
using namespace std;

int printFreeSpace(const char *path)
{
    struct statvfs st;

    if (::statvfs(path, &st) != 0) {
        printf("statvfs error: %s\n", strerror(errno));
        return -1;
    }

    auto freeSize = st.f_bsize * st.f_bfree;
    printf("current free space of path %s: %lu byte\n", path, freeSize);

    return 0;
}

int main()
{
    printFreeSpace("/");
    return 0;
}

```

statvfs, fstatvfs 函数说明
```
有2个接口能获取磁盘信息方式，statvfs需要传入一个C风格Posix路径；fstatvfs需要一个打开的文件描述符。

#include <sys/statvfs.h>
int statvfs(const char *path, struct statvfs *buf);
int fstatvfs(int fd, struct statvfs *buf);

参数说明：

path 要获取磁盘信息的路径，只要是挂载到操作文件系统的路径即可。
fd 是已打开文件描述符，代表要获取磁盘信息的文件。
buf 指向一个statvfs结构，定义如下：
struct statvfs {
    unsigned long  f_bsize;    /* Filesystem block size 文件系统块大小 */
    unsigned long  f_frsize;   /* Fragment size 碎片大小 */
    fsblkcnt_t     f_blocks;   /* Size of fs in f_frsize units  */
    fsblkcnt_t     f_bfree;    /* Number of free blocks 空闲块数量 */
    fsblkcnt_t     f_bavail;   /* Number of free blocks for
                                             unprivileged users 非特权用户的空闲块数量 */
    fsfilcnt_t     f_files;    /* Number of inodes i节点数量 */
    fsfilcnt_t     f_ffree;    /* Number of free inodes 空闲i节点数量 */
    fsfilcnt_t     f_favail;   /* Number of free inodes for
                                             unprivileged users 非特权用户的空闲i节点数量 */
    unsigned long  f_fsid;     /* Filesystem ID 文件系统id */
    unsigned long  f_flag;     /* Mount flags  挂载标识 */
    unsigned long  f_namemax;  /* Maximum filename length 最大文件名长度 */
};

f_flag是bit位掩码的，表示当挂载到该文件系统时的各种选项。包含以下几个选项：

ST_MANDLOCK 文件系统上允许强制锁。
ST_NOATIME 不更新访问时间。
ST_NODEV 不允许访问设备特殊文件。
ST_NODIRATIME 不更新目录访问时间。
ST_NOEXEC 不允许找文件系统上执行程序。
ST_NOSUID 对于文件系统上的可执行文件，exec加载器忽略set-user-ID 和 set-group-ID bit位。
ST_RDONLY 文件系统挂载为只读。
ST_RELATIME 更新访问时间，参考mtime/ctime。
ST_SYNCHRONOUS 写立即同步到文件系统，参见open的O_SYNC选项。
返回值：
成功返回0；失败返回-1，errno被设置。
```

```

#if !defined(AFX_14BEC153_17B9_47BE_845F_71A27BF26B59__INCLUDED_)
#define AFX_14BEC153_17B9_47BE_845F_71A27BF26B59__INCLUDED_
#if _MSC_VER > 1000
#pragma once
#endif // _MSC_VER > 1000
#include <iostream>
#include <string>
#include <windows.h>
	//#define CPUSIZE 17
	using namespace std;
	//--------------------------------------------------------------
	// CPU序列号
	//--------------------------------------------------------------
	BOOL GetCpuByCmd(string &ider, int len = 128);
	BOOL GetDiskByCmd(string &ider, int len = 128);
#endif // !defined(AFX_14BEC153_17B9_47BE_845F_71A27BF26B59__INCLUDED_)
	//--------------------------------------------------------------
	// CPU序列号
	//--------------------------------------------------------------
	BOOL GetCpuByCmd(string &ider, int len/*=128*/)
	{
		//CPU序列
		const long MAX_COMMAND_SIZE = 10000; // 命令行输出缓冲大小
		WCHAR szFetCmd[] = L"wmic cpu get processorid"; // 获取CPU序列号命令行
		const string strEnSearch = "ProcessorId"; // CPU序列号的前导信息
		BOOL bret = FALSE;
		HANDLE hReadPipe = NULL; //读取管道
		HANDLE hWritePipe = NULL; //写入管道
		PROCESS_INFORMATION pi; //进程信息
		STARTUPINFO si; //控制命令行窗口信息
		SECURITY_ATTRIBUTES sa; //安全属性
		char szBuffer[MAX_COMMAND_SIZE + 1] = { 0 }; // 放置命令行结果的输出缓冲区
		string strBuffer;
		unsigned long count = 0;
		long ipos = 0;
		memset(&pi, 0, sizeof(pi));
		memset(&si, 0, sizeof(si));
		memset(&sa, 0, sizeof(sa));
		pi.hProcess = NULL;
		pi.hThread = NULL;
		si.cb = sizeof(STARTUPINFO);

		sa.nLength = sizeof(SECURITY_ATTRIBUTES);
		sa.lpSecurityDescriptor = NULL;
		sa.bInheritHandle = TRUE;
		//1.0 创建管道
		bret = CreatePipe(&hReadPipe, &hWritePipe, &sa, 0);
		if (!bret)
		{
			goto END;
		}
		//2.0 设置命令行窗口的信息为指定的读写管道
		GetStartupInfo(&si);
		si.hStdError = hWritePipe;
		si.hStdOutput = hWritePipe;
		si.wShowWindow = SW_HIDE; //隐藏命令行窗口
		si.dwFlags = STARTF_USESHOWWINDOW | STARTF_USESTDHANDLES;
		//3.0 创建获取命令行的进程
		bret = CreateProcess(NULL, szFetCmd, NULL, NULL, TRUE, 0, NULL, NULL, &si, &pi);
		if (!bret)
		{
			goto END;
		}
		//4.0 读取返回的数据
		WaitForSingleObject(pi.hProcess, 500/*INFINITE*/);
		bret = ReadFile(hReadPipe, szBuffer, MAX_COMMAND_SIZE, &count, 0);
		if (!bret)
		{
			goto END;
		}
		//5.0 查找CPU序列号
		bret = FALSE;
		strBuffer = szBuffer;
		ipos = strBuffer.find(strEnSearch);
		if (ipos < 0) // 没有找到
		{
			goto END;
		}
		else
		{
			strBuffer = strBuffer.substr(ipos + strEnSearch.length());
		}
		memset(szBuffer, 0x00, sizeof(szBuffer));
		strcpy_s(szBuffer, strBuffer.c_str());
		//modify here
		//去掉中间的空格 \r \n
		char temp[512];
		memset(temp, 0, sizeof(temp));
		int index = 0;
		for (size_t i = 0; i < strBuffer.size(); i++)
		{
			if (strBuffer[i] != ' '&&strBuffer[i] != '\n'&&strBuffer[i] != '\r')
			{
				temp[index] = strBuffer[i];
				index++;
			}
		}
		ider = temp;
		bret = TRUE;
	END:
				   //关闭所有的句柄
		CloseHandle(hWritePipe);
		CloseHandle(hReadPipe);
		CloseHandle(pi.hProcess);
		CloseHandle(pi.hThread);
		return(bret);
	}
	//--------------------------------------------------------------
	// 硬盘编号
	//--------------------------------------------------------------
	BOOL GetDiskByCmd(string &ider, int len/*=128*/)
	{
		//diskdrive
		const long MAX_COMMAND_SIZE = 10000; // 命令行输出缓冲大小
		WCHAR szFetCmd[] = L"wmic diskdrive get serialnumber"; // 获取DiskDrive命令行
		const string strEnSearch = "SerialNumber"; // DiskDrive序列号的前导信息
		BOOL bret = FALSE;
		HANDLE hReadPipe = NULL; //读取管道
		HANDLE hWritePipe = NULL; //写入管道
		PROCESS_INFORMATION pi; //进程信息
		STARTUPINFO si; //控制命令行窗口信息
		SECURITY_ATTRIBUTES sa; //安全属性
		char szBuffer[MAX_COMMAND_SIZE + 1] = { 0 }; // 放置命令行结果的输出缓冲区
		string strBuffer;
		unsigned long count = 0;
		long ipos = 0;
		memset(&pi, 0, sizeof(pi));
		memset(&si, 0, sizeof(si));
		memset(&sa, 0, sizeof(sa));
		pi.hProcess = NULL;
		pi.hThread = NULL;
		si.cb = sizeof(STARTUPINFO);
		sa.nLength = sizeof(SECURITY_ATTRIBUTES);
		sa.lpSecurityDescriptor = NULL;
		sa.bInheritHandle = TRUE;
		//1.0 创建管道
		bret = CreatePipe(&hReadPipe, &hWritePipe, &sa, 0);
		if (!bret)
		{
			goto END;
		}
		//2.0 设置命令行窗口的信息为指定的读写管道
		GetStartupInfo(&si);
		si.hStdError = hWritePipe;
		si.hStdOutput = hWritePipe;
		si.wShowWindow = SW_HIDE; //隐藏命令行窗口
		si.dwFlags = STARTF_USESHOWWINDOW | STARTF_USESTDHANDLES;
		//3.0 创建获取命令行的进程
		bret = CreateProcess(NULL, szFetCmd, NULL, NULL, TRUE, 0, NULL, NULL, &si, &pi);
		if (!bret)
		{
			goto END;
		}
		//4.0 读取返回的数据
		WaitForSingleObject(pi.hProcess, 500/*INFINITE*/);
		bret = ReadFile(hReadPipe, szBuffer, MAX_COMMAND_SIZE, &count, 0);
		if (!bret)
		{
			goto END;
		}
		//5.0 查找CPU序列号
		bret = FALSE;
		strBuffer = szBuffer;
		ipos = strBuffer.find(strEnSearch);
		if (ipos < 0) // 没有找到
		{
			goto END;
		}
		else
		{
			strBuffer = strBuffer.substr(ipos + strEnSearch.length());
		}
		memset(szBuffer, 0x00, sizeof(szBuffer));
		strcpy_s(szBuffer, strBuffer.c_str());
		//modify here
		//去掉中间的空格 \r \n
		char temp[512];
		memset(temp, 0, sizeof(temp));
		int index = 0;
		for (size_t i = 0; i < strBuffer.size(); i++)
		{
			if (strBuffer[i] != ' '&&strBuffer[i] != '\n'&&strBuffer[i] != '\r')
			{
				temp[index] = strBuffer[i];
				index++;
			}
		}
		ider = temp;
		bret = TRUE;
	END:
		//关闭所有的句柄
		CloseHandle(hWritePipe);
		CloseHandle(hReadPipe);
		CloseHandle(pi.hProcess);
		CloseHandle(pi.hThread);
		return(bret);
	}
	int main()
	{
		string cpuid;
		if (!GetCpuByCmd(cpuid))
		{
			cout << "get cpu id failed!\n";
			return false;
		}
		cout << "cpu ID: " << cpuid << endl;
		string diskid;
		if (!GetDiskByCmd(diskid))
		{
			cout << "get diskdrive failed!\n";
			return false;
		}
		cout << "disk ID: " << diskid << endl;
	}
```

# 182 strcspn function


```
#include<stdio.h>
#include<string.h>

 int IsCodeLegal(char* code){
	char* rule1="ABCDEFGHIJKLMNOPQRSTUVWXYZ";
	char* rule2="abcdefghijklmnopqrstuvwxyz";
	char* rule3="~!@#$%^&*";
	int result_rule1=0,result_rule2=0,result_rule3=0;
	result_rule1=strcspn(rule1,code);
	result_rule2=strcspn(rule2,code);
	result_rule3=strcspn(rule3,code);
	if(result_rule1==26||result_rule2==26||result_rule3==9)
		return 0;
	else
		return 1;
 }

 int main(){
	char* code1="1234567890";
	char* code2="123@Ab27418";
	printf("%s，%s\n",code1,IsCodeLegal(code1)?"合法":"不合法");
	printf("%s，%s\n",code2,IsCodeLegal(code2)?"合法":"不合法");
 }

```

# 183 查询oracle表空间使用情况


```
select 
b.tablespace_name  --表空间名
,b.m_bytes  --表空间大小
,b.m_bytes-nvl(a.mbytes_free,0) used  --已使用空间
,nvl(a.mbytes_free,0) free --剩余空间
,round(((b.m_bytes-nvl(a.mbytes_free,0))/b.m_bytes),2)*100||'%' pct_used --使用率
from
(select sum(bytes)/(1024*1024) mbytes_free,max(bytes)/(1024*1024) largest,tablespace_name
from sys.dba_free_space group by tablespace_name)a,
(select sum(bytes)/(1024*1024) m_bytes,sum(maxbytes)/(1024*1024) mbytes_max,tablespace_name 
from sys.dba_data_files group by tablespace_name
union all
select sum(bytes)/(1024*1024) m_bytes,sum(maxbytes)/(1024*1024) mbytes_max,tablespace_name 
from sys.dba_temp_files group by tablespace_name)b
where a.tablespace_name (+)= b.tablespace_name order by a.tablespace_name asc
```

```
查看数据文件以及所属表空间的相关信息

SELECT * FROM DBA_DATA_FILES;  --查看数据文件信息
SELECT * FROM DBA_TEMP_FILES;  --查看临时数据文件信息
SELECT * FROM DBA_FREE_SPACE;  --查看表空间剩余空间,每段剩余空间都会有一条记录，如果一个表空间记录过多说明碎片过多
```


# 184  Query postgresql table space



Postgresql自带了pg_default、pg_global这两个表空间

表空间pg_default是用来存储系统目录对象、用户表、用户表index、和临时表、临时表index、内部临时表的默认空间。对应存储目录$PADATA/base/

表空间pg_global是用来存放集群级别的系统字典表(比如pg_database)的空间；对应存储目录$PADATA/global/

```
select spcname from pg_tablespace

select * from pg_tablespace
```

```
SELECT oid,spcname AS "Name",
       pg_catalog.pg_get_userbyid(spcowner) AS "Owner",
	   pg_catalog.pg_tablespace_location(oid) AS "Location" 
	   FROM pg_catalog.pg_tablespace where spcname not like 'pg_%'

  oid | Name         | Owner    | Location
------+--------------+----------+-------------------
 25632| dwtablespace | postgres | /mnt/dw
 28632| dw2          | postgres | /Pgtablespace/dw2

```

```
select a.oid,a.datname as "Name",b.spcname as "Tablespace" 
     FROM pg_catalog.pg_database a JOIN pg_catalog.pg_tablespace b 
	 on a.dattablespace = b.oid where b.spcname  like 'pg_%';
	 
  oid | Name | Tablespace
------+------+--------------
15632 | osm  | pg_default
25632 | dwdb | dwtablespace


```


# 185  postgresql default_tablespace

1. 默认表空间只是针对新建的表和索引而言，
新建数据库时如果不指定表空间名称，新数据库不会使用默认表空间而是继承模板数据库继承其表空间设置；

2. 新建表和索引时，如果参数default_tablespace设置了有效的默认表空间，
这些新建的表和索引会使用default_tablespace设置的默认表空间而不会使用这些表和索引对应的数据库的表空间，
也就是当default_tablespace被设置为非空字符串并且有效，那么它就为没有显式TABLESPACE子句的CREATE TABLE和CREATE INDEX命令提供一个隐式TABLESPACE子句。

3. 通过命令show default_tablespace可以看到参数default_tablespace指向的默认表空间，
   该值为空的话，说明就是默认表空间就是当前数据库的表空间

4. 设置：
   alter system set default_tablespace=newtablespace，
   该参数不需要重启只需要执行select pg_reload_conf()就可以马上生效并且重启后也生效，
   如果是SET default_tablespace=newtablespace则只是在当前会话生效



# 186  postgresql index

## 1. 创建索引 CREATE INDEX

使用 CREATE INDEX 语句可以创建普通索引。

普通索引是最常见的索引类型，用于加速对表中数据的查询。

CREATE INDEX 的语法：
```
CREATE INDEX index_name
ON table_name (column1 [ASC|DESC], column2 [ASC|DESC], ...);
```
CREATE INDEX: 用于创建普通索引的关键字。
index_name: 指定要创建的索引的名称。索引名称在表中必须是唯一的。
table_name: 指定要在哪个表上创建索引。
(column1, column2, ...): 指定要索引的表列名。你可以指定一个或多个列作为索引的组合。这些列的数据类型通常是数值、文本或日期。
ASC和DESC（可选）: 用于指定索引的排序顺序。默认情况下，索引以升序（ASC）排序。
以下实例假设我们有一个名为 students 的表，包含 id、name 和 age 列，我们将在 name 列上创建一个普通索引。
```
CREATE INDEX idx_name ON students (name);
```
上述语句将在 students 表的 name 列上创建一个名为 idx_name 的普通索引，这将有助于提高通过姓名进行搜索的查询性能。

需要注意的是，如果表中的数据量较大，索引的创建可能会花费一些时间，但一旦创建完成，查询性能将会显著提高。

## 2. 修改表结构(添加索引)
我们可以使用 ALTER TABLE 命令可以在已有的表中创建索引。

ALTER TABLE 允许你修改表的结构，包括添加、修改或删除索引。

ALTER TABLE 创建索引的语法：
```
ALTER TABLE table_name ADD INDEX index_name (column1 [ASC|DESC], column2 [ASC|DESC], ...);
```
ALTER TABLE: 用于修改表结构的关键字。

下面是一个实例，我们将在已存在的名为 employees 的表上创建一个普通索引:
```
ALTER TABLE employees  ADD INDEX idx_age (age);
```
上述语句将在 employees 表的 age 列上创建一个名为 idx_age 的普通索引。

## 3. 创建表的时候直接指定
我们可以在创建表的时候，你可以在 CREATE TABLE 语句中直接指定索引，以创建表和索引的组合。

```
CREATE TABLE table_name (
  column1 data_type,
  column2 data_type,
  ...,
  INDEX index_name (column1 [ASC|DESC], column2 [ASC|DESC], ...)
);
```

下面是一个实例，我们要创建一个名为 students 的表，并在 age 列上创建一个普通索引。
```
CREATE TABLE students (
  id INT PRIMARY KEY,
  name VARCHAR(50),
  age INT,
  INDEX idx_age (age)
);
```
在上述实例中，我们在 students 表的 age 列上创建了一个名为 idx_age 的普通索引。


## 4. 删除索引的语法
我们可以使用 DROP INDEX 语句来删除索引。

DROP INDEX 的语法：
```
DROP INDEX index_name ON table_name;
```
DROP INDEX: 用于删除索引的关键字。
index_name: 指定要删除的索引的名称。
ON table_name: 指定要在哪个表上删除索引。
使用 ALTER TABLE 语句删除索引的语法如下：
```
ALTER TABLE table_name
DROP INDEX index_name;
```
ALTER TABLE: 用于修改表结构的关键字。
table_name: 指定要修改的表的名称。
DROP INDEX: 用于删除索引的子句。
index_name: 指定要删除的索引的名称。


以下实例假设我们有一个名为 employees 的表，并在 age 列上有一个名为 idx_age 的索引，现在我们要删除这个索引：
```
DROP INDEX idx_age ON employees;
```
或使用 ALTER TABLE 语句：
```
ALTER TABLE employees
DROP INDEX idx_age;
```
这两个命令都会从 employees 表中删除名为 idx_age 的索引。

**如果该索引不存在，执行命令时会产生错误。因此，在删除索引之前最好确认该索引是否存在，或者使用错误处理机制来处理可能的错误情况。**

## 5. 唯一索引
在 MySQL 中，你可以使用 CREATE UNIQUE INDEX 语句来创建唯一索引。

唯一索引确保索引中的值是唯一的，不允许有重复值。
```
CREATE UNIQUE INDEX idx_email ON employees (email);
```
```
CREATE TABLE employees (
  id INT PRIMARY KEY,
  name VARCHAR(50),
  email VARCHAR(100) UNIQUE
);
```

```
CREATE TABLE table_name (
  column1 data_type,
  column2 data_type,
  ...,
  CONSTRAINT index_name UNIQUE (column1 [ASC|DESC], column2 [ASC|DESC], ...)
);
```
CONSTRAINT: 用于添加约束的关键字。
index_name: 指定要创建的唯一索引的名称。约束名称在表中必须是唯一的。
UNIQUE (column1, column2, ...): 指定要索引的表列名。

## 6. 修改表结构
我们可以使用 ALTER TABLE 命令来创建唯一索引。

ALTER TABLE命令允许你修改已经存在的表结构，包括添加新的索引。
```
ALTER table mytable ADD UNIQUE [indexName] (columnName(length))
```
ALTER TABLE: 用于修改表结构的关键字。
table_name: 指定要修改的表的名称。
ADD CONSTRAINT: 这是用于添加约束（包括唯一索引）的关键字。
index_name: 指定要创建的唯一索引的名称。约束名称在表中必须是唯一的。
UNIQUE (column1, column2, ...): 指定要索引的表列名。你可以指定一个或多个列作为索引的组合。这些列的数据类型通常是数值、文本或日期。
ASC和DESC（可选）: 用于指定索引的排序顺序。默认情况下，索引以升序（ASC）排序。


以下是一个使用 ALTER TABLE 命令创建唯一索引的实例：假设我们有一个名为 employees 的表，包含 id 和 email 列，现在我们想在 email 列上创建一个唯一索引，以确保每个员工的电子邮件地址都是唯一的。
```
ALTER TABLE employees
ADD CONSTRAINT idx_email UNIQUE (email);
```
以上实例将在 employees 表的 email 列上创建一个名为 idx_email 的唯一索引。

**请注意，如果表中已经有重复的 email 值，那么添加唯一索引将会失败。在创建唯一索引之前，你可能需要确保表中的 email 列没有重复的值。**

## 7. 使用ALTER 命令添加和删除索引
有四种方式来添加数据表的索引：

```
ALTER TABLE tbl_name ADD PRIMARY KEY (column_list): 该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL。

ALTER TABLE tbl_name ADD UNIQUE index_name (column_list): 这条语句创建索引的值必须是唯一的（除了NULL外，NULL可能会出现多次）。

ALTER TABLE tbl_name ADD INDEX index_name (column_list): 添加普通索引，索引值可出现多次。

ALTER TABLE tbl_name ADD FULLTEXT index_name (column_list):该语句指定了索引为 FULLTEXT ，用于全文索引。
```


# 187   oracle getColumnListMetaData meta crash

## 1. example
 
```
int main(int argc, char* argv[])
{
    string program = argv[0];
    glogInit();
    Environment *env = NULL;
    Connection *conn = NULL;
    Statement *stmt = NULL;
    ResultSet *rs = NULL;
    int errNum;
    string errMsg;
    string username = "usr1";
    string password = "usr1";
    string connstring = "192.168.0.1:1521/orcl";
    env = Environment::createEnvironment();

    assert(env != NULL);
    conn = env->createConnection(username,password,connstring);
    if(!conn)
    {
        return -1;
    }

    Statement *pStmt = NULL;
    string sqlStmt="select * from USER_TABLES";
    pStmt = conn->createStatement(sqlStmt);
    ResultSet *pRs = pStmt->executeQuery(sqlStmt);

    vector<MetaData> metafield = pRs->getColumnListMetaData();

    while(pRs->next()) {
        for(int col =0;col <metafield.size();col++)
        {
            string reStr = pRs->getString(col + 1).empty() ? "" : pRs->getString(col + 1).c_str();
            cout<<" "<<reStr<<" ";
        }
        cout<<endl;
    }

    pStmt->closeResultSet(pRs);
    env->terminateConnection(conn);
    Environment::terminateEnvironment(env);

    return 0;
}
```

## 2. crash log

```
E0827 17:32:38.492885 30841 oracleconnector.cpp:40] *** Aborted at 1566898358 (unix time) try "date -d @1566898358" if you are using GNU date ***
E0827 17:32:38.494403 30841 oracleconnector.cpp:40] PC: @                0x0 (unknown)
E0827 17:32:38.494635 30841 oracleconnector.cpp:40] *** SIGSEGV (@0x10) received by PID 30841 (TID 0x7feef115ae80) from PID 16; stack trace: ***
E0827 17:32:38.495303 30841 oracleconnector.cpp:40]     @     0x7feef0d605d0 (unknown)
E0827 17:32:38.504227 30841 oracleconnector.cpp:40]     @     0x7feeefc45cd2 kpuhhfre
E0827 17:32:38.511675 30841 oracleconnector.cpp:40]     @     0x7feeeedce5df OCIPHeapFree
E0827 17:32:38.512270 30841 oracleconnector.cpp:40]     @     0x7feef05b2974 oracle::occi::HeapAlloc<>::operator delete()
E0827 17:32:38.512823 30841 oracleconnector.cpp:40]     @     0x7feef05e024f _ZN6oracle4occi12MetaDataImplD9Ev
E0827 17:32:38.513456 30841 oracleconnector.cpp:40]     @     0x7feef05e027a oracle::occi::MetaDataImpl::~MetaDataImpl()
E0827 17:32:38.514011 30841 oracleconnector.cpp:40]     @     0x7feef05e011a oracle::occi::RefCounted::onZeroReferences()
E0827 17:32:38.514832 30841 oracleconnector.cpp:40]     @     0x7feef05e0137 oracle::occi::RefCounted::deleteRef()
E0827 17:32:38.515621 30841 oracleconnector.cpp:40]     @     0x7feef05b8079 _ZN6oracle4occi8ConstPtrINS0_12MetaDataImplEED9Ev
E0827 17:32:38.516194 30841 oracleconnector.cpp:40]     @     0x7feef05b8062 oracle::occi::ConstPtr<>::~ConstPtr()
E0827 17:32:38.516757 30841 oracleconnector.cpp:40]     @     0x7feef05b804d _ZN6oracle4occi3PtrINS0_12MetaDataImplEED9Ev
E0827 17:32:38.517264 30841 oracleconnector.cpp:40]     @     0x7feef05b803e oracle::occi::Ptr<>::~Ptr()
E0827 17:32:38.517736 30841 oracleconnector.cpp:40]     @     0x7feef05e06eb _ZN6oracle4occi8MetaDataD9Ev
E0827 17:32:38.518162 30841 oracleconnector.cpp:40]     @     0x7feef05e06dc oracle::occi::MetaData::~MetaData()
E0827 17:32:38.518338 30841 oracleconnector.cpp:40]     @           0x4128c2 std::_Destroy<>()
E0827 17:32:38.518527 30841 oracleconnector.cpp:40]     @           0x4127aa std::_Destroy_aux<>::__destroy<>()
E0827 17:32:38.518707 30841 oracleconnector.cpp:40]     @           0x41265d std::_Destroy<>()
E0827 17:32:38.518891 30841 oracleconnector.cpp:40]     @           0x4124f1 std::_Destroy<>()
E0827 17:32:38.519076 30841 oracleconnector.cpp:40]     @           0x412239 std::vector<>::~vector()
E0827 17:32:38.519229 30841 oracleconnector.cpp:40]     @           0x411d99 main
E0827 17:32:38.519829 30841 oracleconnector.cpp:40]     @     0x7feeecfe8495 __libc_start_main
E0827 17:32:38.520076 30841 oracleconnector.cpp:40]     @           0x40f5b9 (unknown)
E0827 17:32:38.520521 30841 oracleconnector.cpp:40]     @                0x0 (unknown)
```

根据错误提示，是程序执行完，MetaData在析构时出错;
通过测试
```
void testMeta()
{
    vector<MetaData> metafield = pRs->getColumnListMetaData();
	log("getColumnListMetaData may crash");
}
```
发现，对于特定数据会有crash 的现象;对于常规数据，则正常运行；

**当使用getColumnListMetaData获取表列数时，超过63列则释放该 Vector 会报错。**

有人这么说

```
经过测试，发现是OCCI库版本选用错误导致。在VS2015上使用的sdk目录msvc下的oraocci12.dll，未选用vc14子文件夹中的库。所以在编程时一定要选正确所用开发环
境对应的版本库文件，即使能编译过，也能运行。但是有些不正常的BUG还是会出现的！
```

但**对于列数超过63的数据**， 如果连接的是oraocci12.lib,同样会有崩溃现象；
**可以确定的是oraocci12 库的函数 getColumnListMetaData 有bug**

## 3. solution

在不能更新oraocci 库版本的情况下，如果需要获取列类型，需使用其他方式来达到目的；

```
string sql = "select column_name,data_type,data_length,data_precision,data_scale,nullable "
	"from all_tab_columns where table_name = '";
sql += tableName;
sql += "' and owner = '";
sql += owner;
sql += "' order by column_id";
while (rs->next())
{
	fldName = rs->getString(1);
	aFld.data_type = rs->getString(2);
	if (aFld.data_type == "BLOB" || aFld.data_type == "CLOB" || aFld.data_type == "NCLOB")
	{
		RecordLOB(aFld.data_type, fldName,tableName,owner);
	}
}	
```


# 188. ALTER TABLE ADD or DROP COLUMN

```
--添加一列
ALTER TABLE table_name ADD column_1 DATE NOT NULL;
ALTER TABLE table_name ADD column_2 VARCHAR2(44) DEFAULT '';
ALTER TABLE table_name ADD column_3 number(28,10);
--添加一列
ALTER TABLE table_name
ADD(
column_1 type constraint,--列名 类型 约束
column_2 type constraint,
 ...
 );

 --删除一列
 ALTER TABLE table_name DROP COLUMN column_name;
 --删除多列
 ALTER TABLE table_name DROP(column_1, column_2,...);

```

# 189. 如何判断一个.lib文件是静态库还是动态库的导入库


```
使用VS自带的一个工具 - lib.exe。
打开目录“C:\Program Files\Microsoft Visual Studio 10.0\VC\bin”就会看到这个工具（具体存在位置根据vs安装路径）
运行 lib /list hello.lib
如果输出： hello.obj，则是静态库
如果输出： hello.dll，则是动态库的导入库。
```

```
lib /list [文件名] 显示包含内容是*.dll的是动态链接库，显示*.obj或者*.o是静态库

此功能可以帮助查看lib文件是静调库还是dll的导入库

 
Microsoft 库管理器 (LIB.exe) 创建和管理通用对象文件格式 (COFF) 对象文件库。 LIB 还可用于创建导出文件和引用导出定义的导入库。

 说明
您只能从 Visual Studio 命令提示符处启动此工具。 而不能从系统命令提示符或文件资源管理器中启动此工具。


LIB 创建标准库、导入库和导出文件，在生成程序时可将它们与 LINK 一起使用。 LIB 从命令提示运行。

可在下列几种模式下使用 LIB：
生成或修改 COFF 库
将成员对象提取到文件中
创建导出文件和导入库
这些模式是互斥的；每次只能以一种模式使用 LIB。

```

# 190.  lstrcpyn and strncpy

## 1. example
使用了lstrcpyn来进行 unicode 的字符拷贝，结果发现少拷贝了一个字符，看了下MSDN：
```
TCHAR chBuffer[512];
lstrcpyn(chBuffer, "abcdefghijklmnop", 4);
```
chBuffer的结果为abc, 也就是指定了长度4，拷贝3个字符，同时加一个'/0'字符。

而strncpy（unicode版本为_tcsncpy)则中规中矩的拷贝参数指定的字符数。

## 2. function

这两个函数作用相近，很容易用错。
```
LPTSTR lstrcpyn(
LPTSTR lpString1,
LPCTSTR lpString2,
int iMaxLength
);
```

Specifies the number of TCHAR values to be copied from the string pointed to by lpString2 into the buffer pointed to by lpString1, including a terminating null character. This refers to bytes for
ANSI versions of the function or WCHAR values for Unicode versions.

从lpString2中向lpString1复制**iMaxLength**个字节，包括 **/0**，也就是实际复制 **iMaxLength-1**个字节

```
char *strncpy(
 char *strDest,
 const char *strSource,
 size_t count
);
```

The strncpy function copies the initial count characters of strSource to strDest and returns strDest. If count is less than or equal to
the length of strSource, a null character is not appended automatically to the copied string. If count is greater than the length of
strSource, the destination string is padded with null characters up to length count. The behavior of strncpy is undefined if the source
and destination strings overlap.
从strSource向strDest中复制**count**个字节，如果strSource长度不够，后面的字节用/0补齐


# 191.  oracle vs postgresql

	oracle        |         postgresql
	NUMBER[5]     |           int2
	NUMBER[0]     |           int4
	FLOAT         |           int4
	NUMBER[38/8]  |           float8
	NUMBER[38]    |           float4
	NUMBER[5]     |           bool
	VARCHAR2      |           varchar
	NVARCHAR2     |           varchar
	DATE          |           timestamp
	TIMESTAMP     |           timestamp
	NCLOB         |           bytea
	BLOB          |           bytea
	CLOB          |           bytea
	LONG          |            *
		*          |           text

```
   NVARCHAR2[255]   <----->  varchar[2*255  + 1]
   NVARCHAR2[1000]  <----->  varchar[2*1000 + 1]
```

# 192. check little endian

```
//判断大端小端
static union
{
	char c[4];
	unsigned long lEndian;
} endian_test = { { 'l', '?', '?', 'b' } };
#define ENDIANNESS   ((char)endian_test.lEndian)
#define B_LITTLE_ENDIAN   ('l' == ENDIANNESS)

namespace reverseBytes
{
	
uint16_t Rvb_uint16t(uint16_t value) 
{
	return(value & 0x00FFU) << 8 | (value & 0xFF00U) >> 8;
}

uint32_t Rvb_uint32t(uint32_t value) 
{
	return (value & 0x000000FFU) << 24 | (value & 0x0000FF00U) << 8 |
		(value & 0x00FF0000U) >> 8 | (value & 0xFF000000U) >> 24;
}

uint64_t Rvb_uint64t(uint64_t value) 
{
	return (value & 0x00000000000000FFU) << 56 | (value & 0x000000000000FF00U) << 40 |
		(value & 0x0000000000FF0000U) << 24 | (value & 0x00000000FF000000U) << 8 |
		(value & 0x000000FF00000000U) >> 8 | (value & 0x0000FF0000000000U) >> 24 |
		(value & 0x00FF000000000000U) >> 40 | (value & 0xFF00000000000000U) >> 56;
}

float Rvb_float(float value)
{
	float_conv f1, f2;
	f1.f = value;
	f2.c[0] = f1.c[3];
	f2.c[1] = f1.c[2];
	f2.c[2] = f1.c[1];
	f2.c[3] = f1.c[0];
	return f2.f;
}

double Rvb_double(double value)
{
	uint64_t Temp = 0;
	Temp = Rvb_uint64t(*((uint64_t *)&value));
	return *((double *)&Temp);
}

}


```
## example

```
if (B_LITTLE_ENDIAN)
	nAdFldLen = reverseBytes::Rvb_uint32t(nFldLen);

```

# 193. oracle类型	vs postgresql类型

oracle类型	postgresql类型
date	timestamp
tinestamp	timestamp
long	text
long raw	bytea
clob	text
nclob	text
blob	bytea
bfile	bytea
raw	bytea
decimal	decimal
float	double precision
double precision	double precision
dec	decimal
int	integer
integer	integer
real	real
smallint	smallint
binary_float	double precision
binary_double	double precision
rowid	oid
xmltype	xml
binary_integer	integer
pls_integer	integer
timestamp with time zone	timestamp with time zone
timestamp with local time zone	timestamp with time zone


# 194 联合索引
```
select  * from user_favorite
where
 create_user_id = '1234567'
  and channel_id = 1
  and is_delete = 0
order by
 create_time desc
limit 0, 20;
```
执行计划（EXPLAIN）
```
select_type  | table        | type| possible_keys| key| key_len| ref |rows |filtered Extra
SIMPLE       | user_favorite| ref | idx_create_user_id_goods_id |idx_create_user_id_goods_id| 266| const,const |1 10.0 | Using index condition; Using where; Using filesort
```

```
explain select mpoid::int8  ,  ST_AsBinary(mpshape, 'NDR')  , mpginf_id, mpshapebyte,LU_CODE from postgres.sde_landusepg2 where (LU_FUNCTION LIKE '%居住%' AND LU_CODE LIKE '%R%' )  order by mpoid
```

问题分析
上面的explain的key可以看出，命中了表里唯一的索引
重点是Extra：
• Using index condition：使用了索引下推，5.6的新功能，如果索引包含多个条件，索引过滤一遍再回表查询
• Using where：有字段不在索引上，回表过滤
• Using filesort：需要排序，不一定是文件排序也有可能是内存排序
先不管是文件排序还是内存排序（可通过optimizer_trace分析），但可以大致确定的是，是因为需要排序，影响了整体性能。将order by命令去掉，验证得出与数据量少的用户查询耗时一致。

## 解决方案
因为一定是需要按创建时间排序的，但排序又影响了性能，这个问题看似也没办法解决了，那有没有办法是，查询到的结果集已经不需要排序，可以直接返回呢？
答案是肯定的，按照MySQL常用的B+树索引，索引里面结果已经是排好序的，按照我们的查询条件是create_user_id+channel_id，再加上排序字段create_time，创建联合索引
```
CREATE INDEX user_favorite_cui_ci_ct_IDX USING BTREE 
ON user_favorite (create_user_id,channel_id,create_time);
```
条件create_user_id+channel_id查询后的结果已经是按照create_time排序好的结果集

# 195 ORDER BY 

```
ORDER BY 可能出现 FileSort 的几种情况：
order by 字段混用 ASC 和 DESC 排序方式。
SELECT * 时，如果 order by 排序字段不是主键可能导致 FileSort。
联合索引情况下，order by 多字段排序的字段左右顺序和联合索引的字段左右顺序不一致导致 FileSort。
联合索引情况下，where 字段和 order by 字段的左右顺序和联合索引字段左右顺序或者 where 字段出现范围查询都可能导致 FileSort

```



## 适当加大 sort_buffer_size 排序区，
尽量让排序在内存中完成，而不是通过创建临时表放在文件中进行；当然也不能无限加大 sort_buffer_size 排序区，因为 sort_buffer_size 参数是每个线程独占的，设置过大会导致服务器 SWAP 严重，要考虑数据库活动连接数和服务器内存的大小来适当设置排序区。

## 尽量只使用必要的字段，SELECT 具体的字段名称，
而不是 SELECT * 选择所有字段，这样可以减少排序区的使用，提高 SQL 性能


# 196.监视oracle执行的SQL语句(正在执行，已执行，执行性能查看)

```

 select b.FIRST_LOAD_TIME, b.LAST_ACTIVE_TIME, b.SQL_TEXT,b.SQL_FULLTEXT,b.MODULE,b.PARSING_SCHEMA_NAME
from v$sqlarea b
where b.FIRST_LOAD_TIME between '2024-02-20/10:22:00' and '2024-02-20/18:52:02'
and b.SQL_FULLTEXT LIKE '%LANDUSE%'
order by b.LAST_ACTIVE_TIME

```

1.正在执行的
```
 select a.username, a.sid,b.SQL_TEXT, b.SQL_FULLTEXT 
 from v$session a, v$sqlarea b 
 where a.sql_address = b.address
```
2.执行过的
```
select b.SQL_TEXT,b.FIRST_LOAD_TIME,b.SQL_FULLTEXT
from v$sqlarea b
where b.FIRST_LOAD_TIME between '2019-08-15/01:52:00' and '2020-02-20/13:52:02'
order by b.FIRST_LOAD_TIM
```
（此方法好处可以查看某一时间段执行过的sql，并且 SQL_FULLTEXT 包含了完整的 sql 语句）
--注意格式：2019-08-15/01:52:00 可以使用 2019-08-15/1:52:00 不能使用

3.查找前十条性能差的sql
```
 SELECT * FROM (select PARSING_USER_ID,EXECUTIONS,SORTS,
 COMMAND_TYPE,DISK_READS,sql_text FROM v$sqlare
 order BY disk_reads DESC )where ROWNUM<10 
```

4.查看占io较大的正在运行的 session
```
 SELECT se.sid,se.serial#,pr.SPID,se.username,se.status,
 se.terminal,se.program,se.MODULE,se.sql_address,st.event,st.p1text,si.physical_reads,
 si.block_changes FROM v$session se,v$session_wait st,
 v$sess_io si,v$process pr WHERE st.sid=se.sid AND st.
 sid=si.sid AND se.PADDR=pr.ADDR AND se.sid>6 AND st.
 wait_time=0 AND st.event NOT LIKE '%SQL%' ORDER BY physical_reads DESC ;
```
5.其他

```
 select OSUSER, PROGRAM, USERNAME, SCHEMANAME, B.Cpu_Time, STATUS, B.SQL_TEXT
 from V$SESSION A
 LEFT JOIN V$SQL B
 ON A.SQL_ADDRESS = B.ADDRESS
 AND A.SQL_HASH_VALUE = B.HASH_VALUE
 order by b.cpu_time desc ;
```
6.
```
Select a.Sid,
 a.SERIAL#,
 a.status,
 a.USERNAME, --哪个用户运行的SQL
 d.SPID 进程号,
 b.sql_text SQL内容,
 a.MACHINE 计算机名称,
 a.MODULE 运行方式,
 to_char(cast((c.sofar / totalwork * 100) as decimal(18, 1))) || '%' 执行百分比,
 c.elapsed_seconds 已耗时_秒,
 c.time_remaining 预计剩余_秒,
 cast(c.elapsed_seconds / 60 as decimal(18, 2)) 已耗时_分,
 cast(c.time_remaining / 60 as decimal(18, 2)) 预计剩余_分,
 cast(c.elapsed_seconds / 3600 as decimal(18, 2)) 已耗时_时,
 cast(c.time_remaining / 3600 as decimal(18, 2)) 预计剩余_时
 from v$session a, v$sqlarea b, v$session_longops c, v$process d
where a.sql_hash_value = b.HASH_VALUE
 and a.sid = c.sid(+)
 and a.SERIAL# = c.SERIAL#(+)
 --and to_char(cast((c.sofar / totalwork * 100) as decimal(18, 1))) <> '100'
 and a.PADDR = d.ADDR ORDER BY a.MODULE;

```

# 197 Build pgAudit on Windows

more information 
please visit url: https://github.com/njesp/build_pgaudit_on_windows

pgAudit版本支持的PostgreSQL主要版本：

```
pgAudit v1.6.X is intended to support PostgreSQL 14.
pgAudit v1.5.X is intended to support PostgreSQL 13.
pgAudit v1.4.X is intended to support PostgreSQL 12.
pgAudit v1.3.X is intended to support PostgreSQL 11.
pgAudit v1.2.X is intended to support PostgreSQL 10.
pgAudit v1.1.X is intended to support PostgreSQL 9.6.
pgAudit v1.0.X is intended to support PostgreSQL 9.5.
```

## Build pgAudit against a binary installation of PostgreSQL
The full PostgreSQL build has a lot of automatic generation of build options included. The build of pgaudit is no exception. The instructions below are based on what the full build process generates.

Assumptions below are default locations of PostgreSQL.

1. Create a build-folder somewhere.

2. Copy pgaudit.c to this folder.

3. Open a x64 Native Tools Command Prompt for VS 2017.

4. Go to the folder with pgaudit.c.

5. Create a pgaudit.def file. Put the text below in the file.
```
EXPORTS
  Pg_magic_func
  _PG_init
  auditEventStack DATA
  auditLog DATA
  auditLogCatalog DATA
  auditLogClient DATA
  auditLogLevel DATA
  auditLogLevelString DATA
  auditLogParameter DATA
  auditLogRelation DATA
  auditLogStatementOnce DATA
  auditRole DATA
  pg_finfo_pgaudit_ddl_command_end
  pg_finfo_pgaudit_sql_drop
  pgaudit_ddl_command_end
  pgaudit_sql_drop
```
6. Create a compile_response_file.txt file. Put the text below in the file.
 ```
 /I"C:\Program Files\PostgreSQL\11\include\server"
 /I"C:\Program Files\PostgreSQL\11\include\server\port\win32"
 /I"C:\Program Files\PostgreSQL\11\include\server\port\win32_msvc"
 /I"C:\Program Files\PostgreSQL\11\include"
 /Zi
 /nologo
 /W3
 /WX-
 /diagnostics:classic
 /Ox
 /D WIN32
 /D _WINDOWS
 /D __WINDOWS__
 /D __WIN32__
 /D EXEC_BACKEND
 /D WIN32_STACK_RLIMIT=4194304
 /D _CRT_SECURE_NO_DEPRECATE
 /D _CRT_NONSTDC_NO_DEPRECATE
 /D _WINDLL
 /D _MBCS
 /GF
 /Gm-
 /EHsc
 /MD
 /GS
 /fp:precise
 /Zc:wchar_t
 /Zc:forScope
 /Zc:inline
 /Gd
 /TC
 /wd4018
 /wd4244
 /wd4273
 /wd4102
 /wd4090
 /wd4267
 /FC
 /errorReport:queue
 ```
7.Create a link_response_file.txt file. Put the text below in the file.
```
/ERRORREPORT:QUEUE
/INCREMENTAL:NO
/NOLOGO
"C:\Program Files\PostgreSQL\11\lib\postgres.lib"
/NODEFAULTLIB:libc
/DEF:".\pgaudit.def"
/MANIFEST
/MANIFESTUAC:"level='asInvoker' uiAccess='false'"
/manifest:embed
/DEBUG
/SUBSYSTEM:CONSOLE
/STACK:"4194304"
/TLBID:1
/DYNAMICBASE:NO
/NXCOMPAT
/IMPLIB:".\pgaudit.lib"
/MACHINE:X64
/ignore:4197
/DLL
```
8. Build, using the compile and link commands below.
```
cl /c @.\compile_response_file.txt .\pgaudit.c 
link.exe @.\link_response_file.txt /OUT:".\pgaudit.dll" .\pgaudit.obj
```


# 198 joint strings with  python function 

```
def strLoop( str_list ,tag):
    pre = "E:/Qt/msvc2015_64/bin/moc.exe " ;
    for x in range(len(str_list)):
        file = str_list[x]
        tail = "{0}/{1}.h  -o  {0}/moc_{1}.cpp".format(tag, file)
        tar = pre + tail
        print(tar)
    return

str_list = ["aqmmap","tileloader","wms","wmsmap","wmts","wmtsmap","worldfilemap"]
strLoop(str_list,"map")

```

```
E:/Qt/msvc2015_64/bin/moc.exe map/aqmmap.h  -o  map/moc_aqmmap.cpp
E:/Qt/msvc2015_64/bin/moc.exe map/tileloader.h  -o  map/moc_tileloader.cpp
E:/Qt/msvc2015_64/bin/moc.exe map/wms.h  -o  map/moc_wms.cpp
E:/Qt/msvc2015_64/bin/moc.exe map/wmsmap.h  -o  map/moc_wmsmap.cpp
E:/Qt/msvc2015_64/bin/moc.exe map/wmts.h  -o  map/moc_wmts.cpp
E:/Qt/msvc2015_64/bin/moc.exe map/wmtsmap.h  -o  map/moc_wmtsmap.cpp
E:/Qt/msvc2015_64/bin/moc.exe map/worldfilemap.h  -o  map/moc_worldfilemap.cpp
```

# 199. guess image type by parsing image buffer header

```

std::string GuessImageType(char* pImageBuffer, int nBufferSize)
{
	if (nBufferSize < 10 || pImageBuffer == nullptr)
		return "";

	unsigned char* pBuffer = (unsigned char*)pImageBuffer;

	if (pBuffer[0] == 0xFF && pBuffer[1] == 0xD8)
		return ".jpg";

	if (pBuffer[0] == 0x89 && pBuffer[1] == 0x50 && pBuffer[2] == 0x4E &&
		pBuffer[3] == 0x47 && pBuffer[4] == 0x0D && pBuffer[5] == 0x0A &&
		pBuffer[6] == 0x1A && pBuffer[7] == 0x0A)
		return ".png";

	if (pBuffer[0] == 'B' && pBuffer[1] == 'M')
		return ".bmp";

	if ((pBuffer[0] == 'I' && pBuffer[1] == 'I') ||
		(pBuffer[0] == 'M' && pBuffer[1] == 'M'))
		return ".tif";

	if (pBuffer[0] == 'G' && pBuffer[1] == 'I' && pBuffer[2] == 'F')
		return ".gif";

	return "";
}
```

# 200 . opentopomap url

```
<name>Open Topo Map</name>
<url>https://a.tile.opentopomap.org/$z/$x/$y.png</url>
<zoom max="17"/>
<copyright>Map data: © OpenStreetMap contributors (ODbL), SRTM | Rendering: © OpenTopoMap (CC-BY-SA)</copyright>

```
```
https://basemap.nationalmap.gov/ArcGIS/rest/services/USGSImageryOnly/MapServer/tile/2/1/2

```

# 201. ProcessBuilder  commandArgs

```

    public static boolean isLinuxOS() {
        boolean var0 = false;
        String var1 = "os.name";
        String var2 = System.getProperty(var1);
        var2 = var2.toLowerCase();
        if (var2.contains("window")) {
            var0 = false;
        } else if (var2.contains("linux")) {
            var0 = true;
        }

        return var0;
    }

    public static String getProgramPath() {
        String var0 = null;
        boolean var1 = isLinuxOS();
        String var2;
        if (var1) {
            var2 = "libbase.so";
            String var3 = System.getProperty("java.library.path");
            String[] var4 = var3.split(File.pathSeparator);
            String[] var5 = var4;
            int var6 = var4.length;

            for(int var7 = 0; var7 < var6; ++var7) {
                String var8 = var5[var7];
                File var9 = new File(Paths.get(var8, var2).toString());
                if (var9.isFile() && var9.exists()) {
                    try {
                        var0 = (new File(var9.getCanonicalPath())).getParent();
                    } catch (IOException var12) {
                        var12.printStackTrace();
                    }
                    break;
                }
            }
        } else {

        }

        return var0;
    }

    protected  static void process() throws IOException, InterruptedException {
        String programPath = getProgramPath();
        String toolName = isLinuxOS() ? "Licence" : "Licence.exe";
        String authExePath = java.nio.file.Paths.get(programPath, toolName).toString();
        List<String> commandArgs = new ArrayList<>();
        commandArgs.add(authExePath);
        commandArgs.add("-self");
        commandArgs.add("-check");
        ProcessBuilder processBuilder = new ProcessBuilder();
        processBuilder.environment().put(isLinuxOS() ? "LD_LIBRARY_PATH" : "PATH", programPath + File.separator);
        processBuilder.command(commandArgs);
        //processBuilder.redirectOutput(ProcessBuilder.Redirect.INHERIT);
        System.out.println("开始授权验证......");
        Process p = processBuilder.start();
        p.waitFor();

    }
```


# 202. check character set

## windows
```
0. 查看windows操作系统的默认编码(字符集)_默认编码 
在windows cmd 模式下，输入命令 ： chcp

中文windows下，活动代码页为936，意思是"中国-简体中文(GB2312)"

936   (ANSI/OEM - 简体中文 GBK)

说明：代码页是“字符集编码”的别名，也有人称为“内码表”。

1.临时修改系统编码：
直接输入“chcp 65001”，回车键（Enter键）执行，这时候该窗口编码已经是UTF-8编码了

```
```
下表列出了所有支持的代码页及其国家(地区)或者语言：
代码页 国家(地区)或语言
437 美国
708 阿拉伯文(ASMO 708)
720 阿拉伯文(DOS)
850 多语言(拉丁文 I)
852 中欧(DOS) - 斯拉夫语(拉丁文 II)
855 西里尔文(俄语)
857 土耳其语
860 葡萄牙语
861 冰岛语
862 希伯来文(DOS)
863 加拿大 - 法语
865 日耳曼语
866 俄语 - 西里尔文(DOS)
869 现代希腊语
874 泰文(Windows)
932 日文(Shift-JIS)
936 中国 - 简体中文(GB2312)
949 韩文
950 繁体中文(Big5)
1200 Unicode
1201 Unicode (Big-Endian)
1250 中欧(Windows)
1251 西里尔文(Windows)
1252 西欧(Windows)
1253 希腊文(Windows)
1254 土耳其文(Windows)
1255 希伯来文(Windows)
1256 阿拉伯文(Windows)
1257 波罗的海文(Windows)
1258 越南文(Windows)
20866 西里尔文(KOI8-R)
21866 西里尔文(KOI8-U)
28592 中欧(ISO)
28593 拉丁文 3 (ISO)
28594 波罗的海文(ISO)
28595 西里尔文(ISO)
28596 阿拉伯文(ISO)
28597 希腊文(ISO)
28598 希伯来文(ISO-Visual)
38598 希伯来文(ISO-Logical)
50000 用户定义的
50001 自动选择
50220 日文(JIS)
50221 日文(JIS-允许一个字节的片假名)
50222 日文(JIS-允许一个字节的片假名 - SO/SI)
50225 韩文(ISO)
50932 日文(自动选择)
50949 韩文(自动选择)
51932 日文(EUC)
51949 韩文(EUC)
52936 简体中文(HZ)
65000 Unicode (UTF-7)
65001 Unicode (UTF-8)
```
## for linux 
```
[Unauthorized System] root@Kylin:~# locale
LANG=zh_CN.UTF-8
LANGUAGE=zh_CN:zh
LC_CTYPE="zh_CN.UTF-8"
LC_NUMERIC="zh_CN.UTF-8"
LC_TIME="zh_CN.UTF-8"
LC_COLLATE="zh_CN.UTF-8"
LC_MONETARY="zh_CN.UTF-8"
LC_MESSAGES="zh_CN.UTF-8"
LC_PAPER="zh_CN.UTF-8"
LC_NAME="zh_CN.UTF-8"
LC_ADDRESS="zh_CN.UTF-8"
LC_TELEPHONE="zh_CN.UTF-8"
LC_MEASUREMENT="zh_CN.UTF-8"
LC_IDENTIFICATION="zh_CN.UTF-8"
LC_ALL=

[Unauthorized System] root@Kylin:~# echo $LANG
zh_CN.UTF-8


```

## 查看系统是否安装中文字符集
```


locale -a |grep zh      

如果出现了 zh 开头的，代表安装了中文字符集，直接进行第 4 步就行修改即可。

如果未出现 zh 开头的，则需要安装：
	yum -y groupinstall chinese-support      安装中文字符集
安装完成之后，修改系统字符集即可

```
## 修改系统字符集
```

临时修改(当前终端生效)：
	export LANG="zh_CN.UTF-8"

永久修改：
	echo 'export LANG="zh_CN.UTF-8"'  >> /etc/proflile       将单引号中的语句写入到 /etc/profile 文件
	source /etc/profile      重新加载 profile 文件（使之立即生效）

```

# 203  get character set

```

#ifdef _WIN32
#include "windows.h"
#include "stdio.h"
#else
#include <langinfo.h>
#endif

enum characterset
{
	def = 1,
	ansi = 100,
	gbk  = 936,
	uft8 = 65001
}  ;

characterset get_characterset()
{
	characterset cs = characterset::def;

#ifdef _WIN32
	CPINFOEX cpInfoEx;
	UINT codePage;
	codePage = GetACP();
	GetCPInfoEx(codePage, 0, &cpInfoEx);
	printf("windows系统字符集: %s.\n", cpInfoEx.CodePageName);
	if (codePage == 936)
		cs = characterset::gbk;
#else
	char * pInfo = nl_langinfo(CODESET);
	printf("linux系统字符集: %s.\n", pInfo);
	if (pInfo && 0 == strcmpi("UTF-8", pInfo))
		cs = characterset::utf8;;


#endif
	return cs;
}


```

# 204  RapidOCR-json

```
# RapidOCR-json 构建指南

本文档帮助如何在 Windows x64 上编译 RapidOCR-json 。

本文参考了 RapidAI官方的[编译说明](https://github.com/RapidAI/RapidOcrOnnx/blob/main/BUILD.md) 。

## 1. 前期准备

资源链接后面的(括弧里是版本)，请看清楚。

### 1.1 需要安装的工具：

- [Visual Studio 2019](https://learn.microsoft.com/zh-cn/visualstudio/releases/2019/release-notes) (Community)
- [Cmake](https://cmake.org/download/) (Windows x64 Installer)

## 2. 构建项目

1. 解压 `lib.7z` ，将其中两个文件夹 `onnxruntime-static` 和 `opencv-static` 解压到 `cpp/` 中。（注意，是直接放在 `cpp/opencv-static` ，而不是 `cpp/lib/opencv-static` ！）
2. 动动手指，点击运行 `generate-vs-project.bat` ，静等文件生成。
3. 打开 `build-win-vs2019-x64` ，用vs2019打开 `RapidOcrOnnx` 。
11. F5尝试编译。如果有 `成功*个，失败0个……` 那就成功了。如果一个黑窗口一闪而过，那就正常。
12. 项目根目录的 `models.7z` ，解压，将 `models` 文件夹整个放到 `Release` 目录下。
13. 随便拿一张测试图片，命名为 `test.png` 之类，放到 `Release` 目录下。
14. 命令行启动程序并调试： `RapidOCR-json.exe --models=models --det=ch_PP-OCRv3_det_infer.onnx --cls=ch_ppocr_mobile_v2.0_cls_infer.onnx --rec=ch_PP-OCRv3_rec_infer.onnx  --keys=ppocr_keys_v1.txt --image=test.png`
```

# 205 convert text to UTF8

```
std::string run(std::string blocktext)
{
    std::wstring res =   String2Wstring(blocktext, CP_UTF8);
    std::string resUTF8 = Wstring2String(res);
	return resUTF8;
}
       
```	
```

#include <comdef.h>
#include <string>
#include <windows.h>

// -------方式一---------
std::string Wstring2String(std::wstring wstr)
{
    // support chinese
    std::string res;
    int len = WideCharToMultiByte(CP_ACP, 0, wstr.c_str(), wstr.size(), nullptr, 0, nullptr, nullptr);
    if (len <= 0) {
        return res;
    }
    char* buffer = new char[len + 1];
    if (buffer == nullptr) {
        return res;
    }
    WideCharToMultiByte(CP_ACP, 0, wstr.c_str(), wstr.size(), buffer, len, nullptr, nullptr);
    buffer[len] = '\0';
    res.append(buffer);
    delete[] buffer;
    return res;
}

std::wstring String2Wstring(std::string wstr, UINT nCode) ///CP_ACP
{
    std::wstring res;
    int len = MultiByteToWideChar(nCode, 0, wstr.c_str(), wstr.size(), nullptr, 0);
    if (len < 0) {
        return res;
    }
    wchar_t* buffer = new wchar_t[len + 1];
    if (buffer == nullptr) {
        return res;
    }
    MultiByteToWideChar(nCode, 0, wstr.c_str(), wstr.size(), buffer, len);
    buffer[len] = '\0';
    res.append(buffer);
    delete[] buffer;
    return res;
}
```

# 206. MSB8027: Two or more files with the name of ***.cpp will produce outputs to the same location.

## Question

```
有两个Texture.cpp（一个针对真实环境纹理，另一个针对虚拟目标纹理）分别位于不同的子目录中，然而VC++编译器却不太乐意接受这种情况：

C:\Program Files (x86)\MSBuild\Microsoft.Cpp\v4.0\V120\Microsoft.CppBuild.targets(942,5): warning MSB8027: Two or more files with the name of Texture.cpp will produce outputs to the same location. This can lead to an incorrect build result.  The files involved are src\geometry\Texture.cpp, src\graphics\Texture.cpp.
程序会继续编译，但最后将在包含错误文件或者访问错误类时发生错误！
```
```

其实这个已经不是什么新Bug了，在下面这个Microsoft Connect给出的时间线中就有这个问题，并且原本计划是在Visual Studio 2013 Update 1就该得到解决（我现在在用Update 3然而并没有解决！）：https://connect.microsoft.com/VisualStudio/feedback/details/797460/incorrect-warning-msb8027-reported-for-files-excluded-from-build
```
## solution

解决方法
```

VC++编译源文件时默认全部输出（对象文件）到同一个目录下，遇到同名源文件覆盖前面的同名对象文件。
链接时会出现链接失败的现象 ‘error LNK2001: 无法解析的外部符号’

为了解决这个问题，你可以设置输出路径与源文件路径类似。以下操作在Visual Studio 2013 (Update 3)下适用：

右键项目->属性->配置属性->C/C++->输出文件->对象文件名，将$(IntDir)改为$(IntDir)/%(RelativeDir)/。

设置完毕后，编译阶段输出路径将会把源文件路径考虑进去而不是只考虑源文件名
```

```
两个或多个源文件具有相同的名称，并且生成结果保存在同一个中间目录中。 首先生成的文件的输出会被下一个同名文件覆盖。 发生错误后通常会出现 LNK4042 警告。 合并在不同位置使用相同文件名的项目时，可能会发生此错误。
```

可以通过几种方法解决此问题：
```
如果项目有两个或多个同名的源文件，请为这些文件指定唯一名称。
```
```
如果无法更改文件名，请将每个文件编译到唯一的中间目录。 要设置中间文件位置，请在“解决方案资源管理器”中选择源文件，然后右键单击以打开快捷菜单。 选择“属性”以打开源文件的“属性页”对话框 。 选择“配置属性”>“C/C++”>“输出文件”属性页 。 将“对象文件名”属性从 $(IntDir) 更改为 $(IntDir)%(RelativeDir)。 选择“确定”以保存更改 。
```

# 207. thread 

```
#ifdef HAVE_PTHREAD_H
#include <pthread.h>
#elif defined(_WIN32)
#include <windows.h>
#endif

#define	MAX_ARGC	20
#define TEST_REPEAT_COUNT 500
#ifdef HAVE_PTHREAD_H
static pthread_t tid[MAX_ARGC];
#elif defined(_WIN32)
static HANDLE tid[MAX_ARGC];
#endif

static const unsigned int num_threads = 4;

static void *
thread_specific_data(void *private_data)
{
    xmlThreadParams *params = (xmlThreadParams *) private_data;
    const char *filename = params->filename;
}
#ifdef _WIN32
static DWORD WINAPI
win32_thread_specific_data(void *private_data)
{
    thread_specific_data(private_data);
    return(0);
}
#endif
#endif /* LIBXML_THREADS_ENABLED */

void runThread()
{
#ifdef HAVE_PTHREAD_H
        memset(tid, 0xff, sizeof(*tid)*num_threads);

	for (i = 0; i < num_threads; i++) {
	    ret = pthread_create(&tid[i], NULL, thread_specific_data,
				 (void *) &threadParams[i]);
	    if (ret != 0) {
		perror("pthread_create");
		exit(1);
	    }
	}
	for (i = 0; i < num_threads; i++) {
            void *result;
	    ret = pthread_join(tid[i], &result);
	    if (ret != 0) {
		perror("pthread_join");
		exit(1);
	    }
	}
#elif defined(_WIN32)
        for (i = 0; i < num_threads; i++)
        {
            tid[i] = (HANDLE) -1;
        }

        for (i = 0; i < num_threads; i++)
        {
            DWORD useless;
            tid[i] = CreateThread(NULL, 0,
                win32_thread_specific_data, &threadParams[i], 0, &useless);
            if (tid[i] == NULL)
            {
                perror("CreateThread");
                exit(1);
            }
        }

        if (WaitForMultipleObjects (num_threads, tid, TRUE, INFINITE) == WAIT_FAILED)
            perror ("WaitForMultipleObjects failed");

        for (i = 0; i < num_threads; i++)
        {
            DWORD exitCode;
            ret = GetExitCodeThread (tid[i], &exitCode);
            if (ret == 0)
            {
                perror("GetExitCodeThread");
                exit(1);
            }
            CloseHandle (tid[i]);
        }
#endif /* pthreads */
}


```


# 208. set data dir for vscode 

For case：the valid space in default dir maybe not enough;
Traget: do not use the default dir;
To do:

```
1. create a shortcut for vscode
2. open the property of the shortcut;
3. for target item,set the user data dir,
   eg:
   D:\Program\VSCode-win32-x64\Code.exe --user-data-dir "F:\Cache\vscode"
```

# 209.  exportCsv and  exportExcel

```
   private void exportExcel() {
        FileChooser fileChooser = new FileChooser();
        // 设置对话框标题
        fileChooser.setTitle("保存文件");
        // 设置默认文件扩展名过滤器，这里以txt为例
        FileChooser.ExtensionFilter txtFilter = new FileChooser.ExtensionFilter("Excel文件(*.xlsx)", "*.xlsx");
        fileChooser.getExtensionFilters().add(txtFilter);
        fileChooser.setSelectedExtensionFilter(txtFilter);

        java.io.File selectedFile = fileChooser.showSaveDialog(this.tableView.getScene().getWindow());
        if (selectedFile != null) {
            //System.out.println("Selected file path: " + selectedFile.getAbsolutePath());
            try {
                String fileName = selectedFile.getName();
                int extensionStartIndex = fileName.lastIndexOf('.');
                if (extensionStartIndex != -1) {
                    fileName = fileName.substring(0, extensionStartIndex);
                }
                EasyExcel.write(selectedFile.getAbsolutePath(), Scoring.class).registerWriteHandler(new LongestMatchColumnWidthStyleStrategy()).sheet(fileName).doWrite(this.data);
                MessageBox.information(this.tableView.getScene().getWindow(), "保存成功，路径：\r\n" + selectedFile.getAbsolutePath());
            } catch (Throwable e) {
                MessageBox.error(this.tableView.getScene().getWindow(), e.getMessage());
            }
        }
    }

    private void exportCsv() {
        FileChooser fileChooser = new FileChooser();
        // 设置对话框标题
        fileChooser.setTitle("保存文件");
        // 设置默认文件扩展名过滤器，这里以txt为例
        FileChooser.ExtensionFilter txtFilter = new FileChooser.ExtensionFilter("Csv(*.csv)", "*.csv");
        fileChooser.getExtensionFilters().add(txtFilter);
        fileChooser.setSelectedExtensionFilter(txtFilter);

        java.io.File selectedFile = fileChooser.showSaveDialog(this.tableView.getScene().getWindow());
        if (selectedFile != null) {
            //System.out.println("Selected file path: " + selectedFile.getAbsolutePath());
            try {
                EasyExcel.write(selectedFile.getAbsolutePath(), Scoring.class).excelType(ExcelTypeEnum.CSV).sheet().doWrite(this.data);
                addBOMtoFile(selectedFile.getAbsolutePath());
                MessageBox.information(this.tableView.getScene().getWindow(), "保存成功，路径：\r\n" + selectedFile.getAbsolutePath());
            } catch (Throwable e) {
                MessageBox.error(this.tableView.getScene().getWindow(), e.getMessage());
            }
        }
    }

    private static boolean isWindows() {
        String os = System.getProperty("os.name").toLowerCase();
        return os.contains("windows");
    }

    private static void addBOMtoFile(String src) throws IOException {
        if (!isWindows())
            return;
        File inputFile = new File(src);
        File tempFile = File.createTempFile(java.util.UUID.randomUUID().toString(), ".csv");

        RandomAccessFile raf = new RandomAccessFile(inputFile, "rw");
        FileOutputStream tempfos = new FileOutputStream(tempFile);

        byte[] newData = { (byte) 0xEF, (byte) 0xBB, (byte) 0xBF};
        // 将新字节写入文件头
        raf.seek(0);
        tempfos.write(newData);
        // 读取并复制剩余的原文件内容到临时文件
        byte[] buffer = new byte[1024];
        int bytesRead;
        while ((bytesRead = raf.read(buffer)) != -1) {
            tempfos.write(buffer, 0, bytesRead);
        }
        tempfos.close();
        raf.close();
        // 删除原文件
        inputFile.delete();

        // 将临时文件改名为原文件
        tempFile.renameTo(inputFile);
    }


```

# 210. Copy File by char one by one from source file to target file


gisLONG CopyFile(const char * pszSrcFile, const char * pszDestFile)
{
	FILE*      in_file = NULL;
	FILE*      out_file = NULL;
	char       data[1024] = { 0 };
	size_t     bytes_in = 0;
	size_t     bytes_out = 0;
	long len = 0;

	if (!pszSrcFile || strlen(pszSrcFile) <= 0 || !pszDestFile || strlen(pszDestFile) <= 0)
		return 0;

	if ((in_file = fopen(pszSrcFile, "rb")) == NULL)
	{
		fclose(in_file);
		perror(pszSrcFile);
		return 0;
	}
	if ((out_file = fopen(pszDestFile, "wb")) == NULL)
	{
		fclose(in_file);
		fclose(out_file);
		perror(pszDestFile);
		return 0;
	}


	while ((bytes_in = fread(data, 1, 1024, in_file)) > 0)
	{
		bytes_out = fwrite(data, 1, bytes_in, out_file);
		if (bytes_in != bytes_out)
		{
			perror("Fatal write error.\n");
			return 0;
		}
		len += bytes_out;
		printf("copying file .... %d bytes copy\n", len);
	}

	fclose(in_file);
	fclose(out_file);

	return 1;
}


# 211. shell script 

当window下创建的脚本文件需要用于linux环境时，需要将文件转为unix格式进行保存，而不能默认保存为windows格式；
两者的区别在于 windows格式结束符为 CR LF ,unix格式结束符为LF ,mac格式为CR

CRLF 是 carriage return line feed 的缩写；中文意思是 回车换行。
LF 是 line feed 的缩写，中文意思是换行

```
#!/bin/bash
# 校验第一个参数是否存在且非空
if [ -z "$1" ]; then
    echo "错误：必须提供第一个参数（URL）！" >&1
    exit 1
fi

# 处理 URL 结尾斜杠问题
db_source="${1%/}"  # 移除末尾可能存在的斜杠
export db_source

# 拼接路径（确保格式为 http://www.qq.com/acls）
export AclsPathSrc="${db_source}/acls"

# 打印结果
echo "AclsPathSrc：${AclsPathSrc}"

# 校验第二个参数是否存在且非空
if [ -z "$2" ]; then
    echo "错误：必须提供第二个参数（URL）！" >&2
    exit 1
fi

# 处理 URL 结尾斜杠问题
db_tar="${2%/}"  # 移除末尾可能存在的斜杠
export db_tar

# 拼接路径（确保格式为 http://www.qq.com/acls）
export AclsPathtar="${db_tar}/acls"

# 打印结果
echo "AclsPathtar：${AclsPathtar}"

#!/bin/bash
```


# 212. gitclone.com 

## HTTPS
SSH 可能很慢且失败，使用https 对应url 
SSH 示例： git@github.com:kingsoft/thirdparty.git

HTTPS 示例：https://github.com/kingsoft/thirdparty.git

如果 https 依然很慢，可以加 gitclone.com

## gitcone.com
最近偶然发现一个比较好用的解决方案，是采用http://gitclone.com的加速，这里记录一下。

具体来说，在仓库url中增加gitclone.com的前缀，别的地方不变，

即https://github.com/修改为https://gitclone.com/github.com/，

例如原始的clone命令是:
git clone https://github.com/huggingface/transformers 

替换成下面的命令即可：

git clone https://gitclone.com/github.com/huggingface/transformers

实测基本上能做到1M/s的下载速度。

这种加速目前只支持git clone 和git pull 命令，所以适用于拉取别人代码进行本地查看的应用场景。

# 213  name validator


	#pragma region
	#include <string>
	#include <regex>
	#include <unordered_set>
	#include <algorithm>
	#include <cctype>
	#include <fstream>
	// 命名规范检查类
	class validator {

	public:
		Validator(size_t trancateLen);
	protected:
		std::unordered_set<std::string> m_reserved_keywords;
		std::unordered_set<std::string> m existedNames;
	bool m_bRenameInvalidFld:
		size_t m_maxLen = FLD_MAME_LEN;
	public:
		/**
		* @brief Loads keywords from the specified file.
		* @param filename The path to the file containing keywords to load.
		* @note This is a pure virtual function, must be implemented by derived classes.
		*/
		virtual void Loadkeywords(const std::string& filename) = 0;
		//有效性检查核心逻辑
		bool Validate(std::string& fieldName) {
			return ValidateCore(fieldName, m existedNames);
		}
	//验证字段名的有效性
		virtual bool ValidateCore(std::string& fieldName,
								std::unordered_set<std::string>& existingNames) = 0;

		///rename invalid field
		bool IfRenameInvalidFld();
		void setRenameInvalidFld(bool bRename);
	};
	//字段命名规范检查类
	class Fieldvalidator : public Validator {
	public:
		Fieldvalidator(size_t trancatelen);
		~Fieldvalidator();
	public:
		void Loadkeywords(const std::string& filename);
	//验证字段名的有效性
		virtual bool ValidateCore(std::string& fieldName,
								std::unordered_set<std::string > & existingNames);
	// 批是检查接口
		void Batchcheck(const std::vector<std::string>& fields);
	};

	enum ValidationResult {
		VALID,
		EMPTY_NAME,
		LENGTH_EXCEEDED,
		INVALID_FIRST_CHAR,
		INVALID_CHAR,
		RESERVED_KEYWORD,
		DUPLICATE_MAME
	};

	class Namevalidator : public validator {
	public:
		Namevalidator(size_t trancateLen);
		~Namevalidator();
	private:
		ValidationResult i_validate(
			std::string& name,
			const std::unordered_set<std::string>& existingNames
		);
	public:
		void Loadkeywords(const std::string& filename);
	// 验证字段名的有效性
		virtual bool ValidateCore(std::string& fieldName, std: : unordered_set<std::string>& existingNames);
		static std::string getErrorMessage(validationResult result) {
			switch(result) {

			case EMPTY_NAME:
				return "名称不能为空";
			case LENGTH_EXCEEDED:
				return "名称长度超限";
			case INVALID_FIRST_CHAR:
				return "首字符必须为字母、汉字或下划线";
			case INVALID_CHAR:
				return"包含非法字";
			case RESERVED_KEYWORD:
				return "名称与保留关键字冲突";
			case DUPLICATE_NAME:
				return "名称已存在";
			default:
				return "";
			}
		}
	};

	class MDBFieldvalidator : public Fieldvalidator {

	public:
		MDBFieldvalidator(size_t trancateLen);
		~MDBFieldvalidator();
		void Initkeywords();
	public:
	// Access 保留关键字集合
		void Loadkeywords(std::unordered_set<std::string>& keywords);
	//扩展检查:ArcGIs地理数据库字股特殊规则
		bool ValidateForGeodatabase(std::string& fieldName);
	};
	#pragma endregion


	#pragma region Fieldvalidator
	Validator::validator(size_t trancateLen) :
		m_maxLen(trancateLen),
		m_bRenameInvalidFld(true) {

	}
	bool validator::IfRenameInvalidFld() {
		return m_bRenameInvalidFld;
	}
	void validator::setRenameInvalidFld(bool bRename) {
		m_bRenameInvalidFld = bRename;
	}

	void Fieldvalidator::Loadkeywords(const std::string& filename) {
		std::ifstream file(filename.c str());
		std::string keyword;
		while(file >> keyword) {
			m_reserved_keywords.insert(keyword);
		}

	}
	const std::string INVALID_CHARS = ".[]!\\/\"'?*,;<>:{}-+=()";

	Fieldvalidator::ValidateCore(std::string& fieldName, std: : unordered_set<std::string>& existingNames) {
		const char* pszName = fieldName.c str();
		// 检章 1:空值
		if(fieldName.empty()) {
			LogError("当前为空值", pszName);
			return false;
		}
		// 检查 2:首尾不能有空掐
		if(fieldName.front() == ' ' || fieldName.back() == ' ') {

			LogError("字段:%s,字段名不能以空格开头或结尾", pszName);
	return false:
		}
		//检查 3:首字符必须为字母或下划线

		if(mdb::isvalidrirstchar(fieldName)) {

			yogError("字段:%s,首字符必须为字母,汉字或下划线", pszName);
			return false;
		}

		bool bExsitIllegalchar = false,
			//检查 4:非法字符检测正则表达式
		for(char c : fieldName) {
			if(INVALID_CHARS.find(c) != std::string::npos) {
				LogWanning("字段:%s,包含非法字符%s", pszName, c);
				bExsitIllegalchar = true;
				if(m_bRenameInvalidFld) {
					mdb::sanitizeName(fieldName);
					break;
				} else
					return false;

			}
		}
		// 检查 5:字段长度
		if(fieldName.size() > m_maxLen) {
			LogWanning("字段名长度需在 1-%d 字符之间,字段:%s", m_maxLen, pszName);
			if(m_bRenameInvalidFld) {
				mdb::Trancate(fieldName, m_maxLen);
				else
					return false;
			}
		}
	// 检章 6:是百为保留字(不区分大小写)
		std::string upperName = fieldName;
		std::transform(upperName.begin(), upperName.end(), upperName.begin(), ::toupper);
		if(m_reserved_keywords.count(upperName)) {
			LogWanning("字段:%s,字段名不能为保留关键字", pszName);
			if(m_bRenameInvalidFld) {

				mdb::Addsuffix(upperName);
				LogWanning("字段:%s,已添加后缀重命名为:%s", pszName, upperName.c str());
				fieldName = upperName;
			} else
				return false;
		}
	///检查 7:唯一性不区分大小写)
		if(existingNames.count(upperName)) {

			LogWanning("字段:%s,字段名已存在", pszName);
			if(m_bRenameInvalidFld) {
				mdb::Addsuffix(upperName)i
				LogWanning("字段:%s,已添加后缀重命名为:%s", pszName, upperName.c_str());
				fieldName = upperName;
			} else
				return false;

		}
		existingNames.insert(upperName);
		return true;
	}
	void Fieldvalidator::Batchcheck(const std::vector<std::string>& fields) {

		for(const auto& field : fields) {
			std::string fld = field;
			bool isvalid = validate(fld);
			std::cout << "字段'" << field << "' :"
					<< (isvalid ? "有效" : "无效") << std::endl;
		}
	}
	Fieldvalidator::Fieldvalidator(size_t trancateLen):
		Validator(trancateLen)

	{

	}
	Fieldvalidator::~Fieldvalidator() {

	}

	#pragma endregion

	namespace mdb {

	//名称枚举,用于指定验证场景
	enum validType {

		validFieldName = 1,
		validsFclsName,
		validAclsName,
		validoclsName,
		validFdsName,
		validAliasName
	};
	enum logLevel {

		debug = 0,
		info = 1,
		warn = 2,
		error = 4,
		fatal = 8
	};

	//验证给定字符串是否符合指定类型的规则
	bool Validate(std::string& str, validType type);
	//替换非法字符为下划线,确保名称的合法性
	int SanitizeName(std::string& name);
	//合名称添加后缀,以避免命名冲突
	int Addsuffix(std::string& name);
	// 截断过长的名称,使其符合最大长度限制
	void Trancate(std::string& sanName, size_t maxLen);
	//判断字符串是否以GBK编码的汉字开头
	bool startswithchinese(const std::string &s);
	//验证字符串是否为有效的首字符,用于命名规范性检查
	bool IsvalidFirstchar(const std::string &s);
	}

	#pragma region Namevalidator
	Namevalidator::Namevalidator(size_t trancateLen):
		Validator(trancateLen) {

	}
	Namevalidator::~Namevalidator()
	{}
	void NameValidator::Loadkeywords(const std::string& filename) {

		std::unordered_set<std::string>keywords = {
			"SELECT", "INSERT", "UPDATE", "DELETE", "TABLE", "OBJECTID", "SHAPE"
		};
		for(std::string keyword : keywords) {
			m_reserved_keywords.insert(keyword);
		}
	}

	bool Namevalidator::ValidateCore(std::string& fieldName, std::unordered_set<std::string>& existingNames) {
		ValidationResult result = i_validate(fieldName, existingNames),
		if(result != VALID) {
			std::string strMsg = Namevalidator::geterrorMessage(result);
			Lognrning("Msg:%s,名称:%s", strMsg.c str(), fieldName.c str());
		}
		if(result == VALID) return true;
		else if(resuIt == LENGTH_EXCEEDED) {

			if(m_bRenameInvalidFld)
				mdb::Trancate(fieldName, m_maxLen);
			return true;
		} 
		else if(result == INVALID_CHAR) {
			if(m_bRenameInvalidFld) {
				mdb::sanitizeName(fieldName);
			}
			return true;
		} 
		else if(result == RESERVED_KEYWORD || resuIt == DUPLICATE_NAME) {

			if(m_bRenameInvalidFld) {
				mdb::Addsuffix(fieldName);
			}
			return true;
		} 
		else return false;

	}

	ValidationResult Namevalidator::i_validate(std::string& name, const std: : unordered_set<std::string>& existingNames) {

	// 检查空值
		if(name.empty()) return EMPTY_NAME;
	// 检查长度
		size_t maxLength = m_maxLen;
		if(name.length() > maxLength) return LENGTH_EXCEEDED;
	//   检查首字符
		if(mdb::IsvalidFirstchar(name))return INVALID_FIRST_CHAR;
	// 检查非法字符
		for(char c : name) {
			if(INVALID_CHARs.find(c) != std::string::npos) {
				return INVALID_CHAR;
			}
		}
	//std::regex pattern(R"(^[A-za-28-9 +)");
	//if(!std::regex match(name, pattern))return INVALID_CHAR;
	// 检查保留字,(不区分大小写)
		std::string upperName = name;
		std::transform(upperName.begin(), upperName.end(), upperName.begin(), ::toupper);
		if(m reserved KeyNords, count(upperName))return RESERVED_KEYWORD;
	// 检查准一性(企业库不区分大小写)
		std::string lookupName = upperName;
		if(existingNames.count(lookupName))return DUPLICATE_NAME;
		return VALID;
	}

	#pragma endregion

	#pragma region MDBFieldValidator
	MDBFieldValidator::MDBFieldValidator(size_t trancateLen):
		FieldValidator(trancateLen) {

		InitKeywords();
	}
	MDBFieldValidator::~MDBFieldValidator()
	{}
	void MDBFieldValidator::InitKeywords() {
		std::unordered_set<std::string>keyWords = {
			"ADD", "ALL", "ALTER",  "AND", "AS", "ASC",
			"BETWEEN", "BY", "COLUMN", "GROUP", "CREATE", "DELETE", "DESC", "DROP", "EXISTS", "FROM",
			"IN", "LIKE", "NOT", "NULL", "OR", "ORDER', "INSERT", 'INTO",
			"JOIN", "KEY", "PRIMARY",
			"SELECT", "TABLE", "UPDATE", "VALUES", "WHERE", "CLASS"
		};
		LoadKeywords(keywords);
	}
	}
	void MDBFieldValidator::LoadKeywords(std::unordered_set<std::string>& keyWords) {
		for(std::string keyword : keyWords) {
			m reserved Keylords.insert(keyword);
		}
	}

		bool MDBFieldValidator::ValidateForGeodatabase(std::string& fieldName) {
			//基础语法检育
			if(!FieldValidator::Validate(fieldName))return false;
	//地理数据库字段禁用前缀(如shape、objectid)
			if(fieldName.find("shape_") == 0 ||
					fieldName.find("objectid") != std::string::npos) {
				return false;
			}
			return true;
		}

	#Pragma endregion
	bool mdb::Validate(std::string& str, validType type)
		{
		bool rtn = false;
		switch(type) {
		case validFieldName : {

			MDBFieldValidator fldValidator(FLD_NAME_LEN);
			rtn = fldValidator.Validate(str);
			break;
		}
		case validsFclsName: {

			NameValidator clsValidator(50);//52
			rtn = clsValidator.Validate(str);
			break;
		}
		case validAclsName:
		case validOclsName:
		case validFdsName: {

			NameValidator clsValidator(60);//64
			rtn = clsValidator.Validate(str);
			break;
		}
		case validAliasName: {

			NameValidator aliasValidator(MAX_ALIAS_LEN - 3);
			rtn = aliasValidator.Validate(str);
			break;
		}
		default:
			break;
			return rtn;
		}

	int mdb::SanitizeName(std::string& name) {
		const char* pszName = name.c str();
		std::replace_if(name.begin(), name.end()
			[](char c) { return INVALID_CHARs.find(c) != std::string::npos; }, '_');
		Logwanning("已重命名,%s-->%s", pszName, name.c_str());
		return 1;

	}
	int mdb::Addsuffix(std::string& name) {

	/// add suffix with gdb
	//return 1;
		std::string strName(name);
		int index = 1;
		name += "_";
		name += std::to string(index);
		yogwanning("已添加后缀重命名,%5-->%s", strName.c str(), name.c str());
		return index;
	}

	void mdb::Trancate(std::string& sanName, size_t maxLen) {
		const char* pszName = sanName.c str();
		if(maxLen <= FLD_NAME_LEN)
			sanName = sanName.substr(0, maxLen);
		else {

			size_t len = sanName.size();
			size_t lhalf = maxLen / 2;
			size_t lpos = len - lhalf;
			if(lpos < 0) lpos = 0;
			sanName = sanName.substr(0, lhalf) + sanName.substr(lpos, lhalf);
		}
		Logwarning("已截断重命名,%s-->%s", pszName, sanName.c str());
	}

	bool mdb::startsWithchinese(const std::string &s) {
		if(s.size() < 2)return false;
		unsigned char c1 = s0], c2 = s1];
	//GBK汉字的首字节0x81-0xFE,次字节0x40-0xFE
		return(c1 >= 0x81 && c1 <= 0xFE) && (c2 >= 0x40 && c2 <= BxFE);
	}
	bool mdb::IsValidFirstchar(const std::string &fieldName)
	{
		return !std::isalpha(fieldName[0l) && fieldName[0l != '_' && !mdb::startswithchinese(fieldName);
	}


# 214 build libpqxx

## install postgresql firstly
## build libpqxx
1.download libpqxx 
2.
  cmake -G"Visual Studio 17 2022" -A x64 -DPostgreSQL_ROOT="E:/PostgreSQL13" -DCMAKE_BUILD_TYPE="Release" -DCMAKE_CONFIGURATION_TYPES-"Release" -DBUILD_SHARED_LIBS=ON -S ../


'''
Administrator@WIN-55826SEKP4A MINGW64 /g/sql/libpqxx-7.10.1
$ mkdir build

Administrator@WIN-55826SEKP4A MINGW64 /g/sql/libpqxx-7.10.1
$ cd build

Administrator@WIN-55826SEKP4A MINGW64 /g/sql/libpqxx-7.10.1/build
$ cmake -G"Visual Studio 17 2022" -A x64 -DPostgreSQL_ROOT="E:/PostgreSQL13" -DCMAKE_BUILD_TYPE="Release" -DCMAKE_CONFIGURATION_TYPES-"Release" -DBUILD_SHARED_LIBS=ON
CMake Warning:
  No source or binary directory provided.  Both will be assumed to be the
  same as the current working directory, but note that this warning will
  become a fatal error in future CMake releases.


CMake Error: Parse error in command line argument: CMAKE_CONFIGURATION_TYPES-Release
 Should be: VAR:type=value

CMake Error: Run 'cmake --help' for all supported options.

Administrator@WIN-55826SEKP4A MINGW64 /g/sql/libpqxx-7.10.1/build
$ cmake --help
Usage

  cmake [options] <path-to-source>
  cmake [options] <path-to-existing-build>
  cmake [options] -S <path-to-source> -B <path-to-build>

Specify a source directory to (re-)generate a build system for it in the
current working directory.  Specify an existing build directory to
re-generate its build system.

Options
  -S <path-to-source>          = Explicitly specify a source directory.
  -B <path-to-build>           = Explicitly specify a build directory.
  -C <initial-cache>           = Pre-load a script to populate the cache.
  -D <var>[:<type>]=<value>    = Create or update a cmake cache entry.
  -U <globbing_expr>           = Remove matching entries from CMake cache.
  -G <generator-name>          = Specify a build system generator.
  -T <toolset-name>            = Specify toolset name if supported by
                                 generator.
  -A <platform-name>           = Specify platform name if supported by
                                 generator.
  --toolchain <file>           = Specify toolchain file
                                 [CMAKE_TOOLCHAIN_FILE].
  --install-prefix <directory> = Specify install directory
                                 [CMAKE_INSTALL_PREFIX].
  -Wdev                        = Enable developer warnings.
  -Wno-dev                     = Suppress developer warnings.
  -Werror=dev                  = Make developer warnings errors.
  -Wno-error=dev               = Make developer warnings not errors.
  -Wdeprecated                 = Enable deprecation warnings.
  -Wno-deprecated              = Suppress deprecation warnings.
  -Werror=deprecated           = Make deprecated macro and function warnings
                                 errors.
  -Wno-error=deprecated        = Make deprecated macro and function warnings
                                 not errors.
  --preset <preset>,--preset=<preset>
                               = Specify a configure preset.
  --list-presets[=<type>]      = List available presets.
  --workflow [<options>]       = Run a workflow preset.
  -E                           = CMake command mode.  Run "cmake -E" for a
                                 summary of commands.
  -L[A][H]                     = List non-advanced cached variables.
  -LR[A][H] <regex>            = Show cached variables that match the regex.
  --fresh                      = Configure a fresh build tree, removing any
                                 existing cache file.
  --build <dir>                = Build a CMake-generated project binary tree.
                                 Run "cmake --build" to see compatible
                                 options and a quick help.
  --install <dir>              = Install a CMake-generated project binary
                                 tree.  Run "cmake --install" to see
                                 compatible options and a quick help.
  --open <dir>                 = Open generated project in the associated
                                 application.
  -N                           = View mode only.
  -P <file>                    = Process script mode.
  --find-package               = Legacy pkg-config like mode.  Do not use.
  --graphviz=<file>            = Generate graphviz of dependencies, see
                                 CMakeGraphVizOptions.cmake for more.
  --system-information [file]  = Dump information about this system.
  --print-config-dir           = Print CMake config directory for user-wide
                                 FileAPI queries.
  --log-level=<ERROR|WARNING|NOTICE|STATUS|VERBOSE|DEBUG|TRACE>
                               = Set the verbosity of messages from CMake
                                 files.  --loglevel is also accepted for
                                 backward compatibility reasons.
  --log-context                = Prepend log messages with context, if given
  --debug-trycompile           = Do not delete the try_compile build tree.
                                 Only useful on one try_compile at a time.
  --debug-output               = Put cmake in a debug mode.
  --debug-find                 = Put cmake find in a debug mode.
  --debug-find-pkg=<pkg-name>[,...]
                               = Limit cmake debug-find to the
                                 comma-separated list of packages
  --debug-find-var=<var-name>[,...]
                               = Limit cmake debug-find to the
                                 comma-separated list of result variables
  --trace                      = Put cmake in trace mode.
  --trace-expand               = Put cmake in trace mode with variable
                                 expansion.
  --trace-format=<human|json-v1>
                               = Set the output format of the trace.
  --trace-source=<file>        = Trace only this CMake file/module.  Multiple
                                 options allowed.
  --trace-redirect=<file>      = Redirect trace output to a file instead of
                                 stderr.
  --warn-uninitialized         = Warn about uninitialized values.
  --no-warn-unused-cli         = Don't warn about command line options.
  --check-system-vars          = Find problems with variable usage in system
                                 files.
  --compile-no-warning-as-error= Ignore COMPILE_WARNING_AS_ERROR property and
                                 CMAKE_COMPILE_WARNING_AS_ERROR variable.
  --profiling-format=<fmt>     = Output data for profiling CMake scripts.
                                 Supported formats: google-trace
  --profiling-output=<file>    = Select an output path for the profiling data
                                 enabled through --profiling-format.
  -h,-H,--help,-help,-usage,/? = Print usage information and exit.
  --version,-version,/V [<file>]
                               = Print version number and exit.
  --help <keyword> [<file>]    = Print help for one keyword and exit.
  --help-full [<file>]         = Print all help manuals and exit.
  --help-manual <man> [<file>] = Print one help manual and exit.
  --help-manual-list [<file>]  = List help manuals available and exit.
  --help-command <cmd> [<file>]= Print help for one command and exit.
  --help-command-list [<file>] = List commands with help available and exit.
  --help-commands [<file>]     = Print cmake-commands manual and exit.
  --help-module <mod> [<file>] = Print help for one module and exit.
  --help-module-list [<file>]  = List modules with help available and exit.
  --help-modules [<file>]      = Print cmake-modules manual and exit.
  --help-policy <cmp> [<file>] = Print help for one policy and exit.
  --help-policy-list [<file>]  = List policies with help available and exit.
  --help-policies [<file>]     = Print cmake-policies manual and exit.
  --help-property <prop> [<file>]
                               = Print help for one property and exit.
  --help-property-list [<file>]= List properties with help available and
                                 exit.
  --help-properties [<file>]   = Print cmake-properties manual and exit.
  --help-variable var [<file>] = Print help for one variable and exit.
  --help-variable-list [<file>]= List variables with help available and exit.
  --help-variables [<file>]    = Print cmake-variables manual and exit.

Generators

The following generators are available on this platform (* marks default):
* Visual Studio 17 2022        = Generates Visual Studio 2022 project files.
                                 Use -A option to specify architecture.
  Visual Studio 16 2019        = Generates Visual Studio 2019 project files.
                                 Use -A option to specify architecture.
  Visual Studio 15 2017 [arch] = Generates Visual Studio 2017 project files.
                                 Optional [arch] can be "Win64" or "ARM".
  Visual Studio 14 2015 [arch] = Generates Visual Studio 2015 project files.
                                 Optional [arch] can be "Win64" or "ARM".
  Borland Makefiles            = Generates Borland makefiles.
  NMake Makefiles              = Generates NMake makefiles.
  NMake Makefiles JOM          = Generates JOM makefiles.
  MSYS Makefiles               = Generates MSYS makefiles.
  MinGW Makefiles              = Generates a make file for use with
                                 mingw32-make.


Administrator@WIN-55826SEKP4A MINGW64 /g/sql/libpqxx-7.10.1/build
$

Administrator@WIN-55826SEKP4A MINGW64 /g/sql/libpqxx-7.10.1/build
$ cmake -G"Visual Studio 17 2022" -A x64 -DPostgreSQL_ROOT="E:/PostgreSQL13" -DCMAKE_BUILD_TYPE="Release" -DCMAKE_CONFIGURATION_TYPES-"Release" -DBUILD_SHARED_LIBS=ON -S ../
CMake Error: Parse error in command line argument: CMAKE_CONFIGURATION_TYPES-Release
 Should be: VAR:type=value

CMake Error: Run 'cmake --help' for all supported options.

Administrator@WIN-55826SEKP4A MINGW64 /g/sql/libpqxx-7.10.1/build
$ cmake -G"Visual Studio 17 2022" -A x64 -DPostgreSQL_ROOT="E:/PostgreSQL13" -DCMAKE_BUILD_TYPE="Release" -DCMAKE_CONFIGURATION_TYPES="Release" -DBUILD_SHARED_LIBS=ON
CMake Warning:
  No source or binary directory provided.  Both will be assumed to be the
  same as the current working directory, but note that this warning will
  become a fatal error in future CMake releases.


CMake Error: The source directory "G:/sql/libpqxx-7.10.1/build" does not appear to contain CM
akeLists.txt.
Specify --help for usage, or press the help button on the CMake GUI.

Administrator@WIN-55826SEKP4A MINGW64 /g/sql/libpqxx-7.10.1/build
$ cmake -G"Visual Studio 17 2022" -A x64 -DPostgreSQL_ROOT="E:/PostgreSQL13" -DCMAKE_BUILD_TYPE="Release" -DCMAKE_CONFIGURATION_TYPES="Release" -DBUILD_SHARED_LIBS=ON -S ../
-- Selecting Windows SDK version 10.0.20348.0 to target Windows 10.0.19045.
-- The CXX compiler identification is MSVC 19.43.34810.0
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: E:/program/vs2022/Enterprise/VC/Tools/MSVC/14.43.34808/bin/Hostx6
4/x64/cl.exe - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Found PostgreSQL: E:/PostgreSQL13/lib/libpq.lib (found version "13.21")
-- Looking for poll
-- Looking for poll - not found
-- Generating config.h
-- Generating config.h - done
-- Configuring done (28.1s)
-- Generating done (0.1s)
-- Build files have been written to: G:/sql/libpqxx-7.10.1/build

'''

# 215 restful Spring Boot

'''
package com.example.zonex;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.util.concurrent.atomic.AtomicLong;

// 假设外部JAR中的类
class ExternalCalculator {
    public int add(int a, int b) {
        return a + b;
    }
}

@RestController
public class CalculatorService {
    private final ExternalCalculator calculator = new ExternalCalculator();

    /// http://localhost:8080/add?a=5&b=3
    @GetMapping("/add")
    public int add(@RequestParam int a, @RequestParam int b) {
        return calculator.add(a, b);
    }

    private static final String template = "Hello, %s!";
    private final AtomicLong counter = new AtomicLong();

    @GetMapping("/greeting")
    public Greeting greeting(@RequestParam(value = "name", defaultValue = "World") String name) {
        return new Greeting(counter.incrementAndGet(), String.format(template, name));
    }

    ///  http://localhost:8080/hi
    @RequestMapping("/hi")
    public String hi(@RequestParam(value = "name", defaultValue = "World") String name) {
        return String.format(template, name);
    }

    /// http://localhost:8080/hello
    @RequestMapping("/hello")
    public Greeting hello(@RequestParam(value = "name", defaultValue = "World") String name) {
        return new Greeting(counter.incrementAndGet(), String.format(template, name));
    }
}
'''



'''

PS C:\.jdks\openjdk-24.0.1\bin> .\java.exe -jar G:\restservice\target\restservice-0.0.1-SNAPSHOT.jar --server.port=8015

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/

 :: Spring Boot ::                (v3.5.3)

'''

# 216 TopoJson

https://geojson.cn/api/china/1.6.2/china.topo.json
https://file.geojson.cn/china/1.6.2/350000.topo.json  福州
https://geojson.cn/api/china/1.6.2/330000.topo.json   浙江
https://geojson.cn/api/china/1.6.2/330000/330100.topo.json 杭州

https://geojson.cn/api/china/1.6.2/420000.topo.json 湖北
https://geojson.cn/api/china/1.6.2/420000/420100.topo.json 武汉
https://geojson.cn/api/china/1.6.2/420000/420700.topo.json 鄂州
https://geojson.cn/api/china/1.6.2/410000.topo.json 河南
https://geojson.cn/api/china/1.6.2/410000/410100.topo.json 郑州
https://geojson.cn/api/china/1.6.2/410000/411000.topo.json 


# 217  compile gdal and proj in linux  

## 1 编译proj


 ./configure --without-curl --prefix=/data/devops/proj --exec-prefix=/data/devops/proj/bin SQLITE3_CFLAGS="-I/data/sdk/3rd/sqlite3/include" SQLITE3_LIBS="-L/data/devops/desktop/program -lsqlite" --enable-tiff=no CFLAGS="-fstack-protector-all -fPIC" LDFLAGS="-Wl,-z,now  -Wl,-z,relro -Wl,-z,noexecstack -Wtrampolines -s"




## 2 编译gdal

arm机器上的cmake版本太低，导致无法找到gdal，将cmake3.25.0版本中的findGDAL.cmake文件更换到arm机器上后，使用下面命令编译



 cmake -G "Unix Makefiles" -DCMAKE_CXX_FLAGS_RELEASE="-fPIC -fstack-protector-all -std=c++11" -DCMAKE_INSTALL_PREFIX=/data/devops/libLAS-1.8.1/ -DCMAKE_BUILD_TYPE=RELEASE -DWITH_GDAL=TRUE -DWITH_GEOTIFF=TRUE -DWITH_LASZIP=FALSE -DWITH_PKGCONFIG=TRUE -DWITH_STATIC_LASZIP=FALSE -DWITH_TESTS=FALSE -DWITH_UTILITIES=FALSE -DBUILD_OSGEO4W=FALSE -DFindGDAL_SKIP_GDAL_CONFIG=TRUE -DGDAL_INCLUDE_DIR=/data/devops/gdal3.0.1/bin/include -DGDAL_LIBRARY=/data/devops/desktop/program/libgdal.so -DBoost_NO_BOOST_CMAKE=ON -DBoost_INCLUDE_DIR=/data/sdk/3rd/boost/include -DBOOST_LIBRARYDIR=/data/devops/desktop/program -DGEOTIFF_INCLUDE_DIR=/data/sdk/3rd/geotiff/include -DGEOTIFF_LIBRARY=/data/devops/desktop/program/libgeotiff.so -DPROJ4_INCLUDE_DIR=/data/devops/PROJ-8.0.1/bin -DPROJ4_LIBRARY=/data/devops/desktop/program/libproj.so -DTIFF_INCLUDE_DIR=/data/sdk/3rd/tiff/include -DTIFF_LIBRARY=/data/devops/desktop/program/libtiff.so -DJPEG_INCLUDE_DIR=/data/sdk/3rd/jepg/include -DJPEG_LIBRARY=/data/devops/desktop/program/libjpeg.so


可能执行的过程中报如下错误：
gt_wkt_srs.cpp中有FixupOrdering()函数找不到：在提示的位置屏蔽掉FixupOrdering函数调用即可；

gt_citation.cpp中GetPrimeMeridian(NULL)，GetAngularUnits(NULL)调用的接口不明确：在对应位置将NULL改为nullptr即可。




# 218  download data from osm

https://download.geofabrik.de/asia/china/beijing-latest-free.shp.zip
https://download.geofabrik.de/asia/china/guangdong-latest-free.shp.zip
https://download.geofabrik.de/asia/china-latest.osm.pbf
https://download.geofabrik.de/asia/taiwan-latest-free.shp.zip


# 219 need by /home/lib.so, try using -rpath or rpath-link

https://stackoverflow.com/questions/10188173/how-to-add-a-directory-to-the-library-path-in-gcc
 ## 问题原因​​
•
​​链接器默认路径​​：仅搜索标准路径（如 /usr/lib、/lib），无法定位自定义路径 /home/lib.so。

•
​​动态库依赖​​：程序运行时需加载共享库（.so），但未指定其位置

## 解决方案

​1. 使用 -rpath（链接+运行阶段）​​
-rpath将路径​​嵌入可执行文件​​，供运行时加载器使用：

gcc -o myapp main.o -L/home -l:lib.so -Wl,-rpath=/home
•
​​效果​​：程序运行时自动在 /home查找 lib.so。

✅ 2. 设置 LD_LIBRARY_PATH（临时运行时）​​
export LD_LIBRARY_PATH=/home:$LD_LIBRARY_PATH  # 临时生效
./myapp
•
​​限制​​：需每次运行前设置，不推荐长期使用。

## 3. 使用 -rpath-link（链接阶段）​​
-rpath-link仅用于链接阶段，不嵌入可执行文件，仅用于链接器查找依赖库：


# 220 scp linux  使用方法

## scp 命令 

scp 命令是 SSH 提供的一个客户端程序，用来远程拷贝文件。scp 命令可以在 Linux 主机之间复制文件和目录，也可以从本地系统复制文件到远程系统，或者从远程系统复制文件到本地系统，同时支持增量备份。

scp 命令的基本语法如下所示：
	
	scp [OPTION] [file] [user@host:] [file]
		
scp 命令的常用选项如下所示：
	
	-r 递归复制整个目录。
	-q 静默模式，不显示进度信息。
	-C 启用压缩。
	-l 限制带宽，单位是 Kbit/s。
	-v 显示进度信息。
	-S 指定加密传输时使用的程序。
	-i 指定私钥文件。
	-o 指定 SSH 选项。
	-F 指定 SSH 配置文件。
	-B 使用批处理模式。
	-P 不是端口号，而是指定是否显示输出。	

## 从本地系统复制文件到远程系统
	
	scp [OPTION] [file] [user@host:] [file]		
	
例如，将本地系统中的 /home/user1/file.txt 文件复制到远程系统中的 /home/user2 目录下，可以使用如下命令：
	
	scp /home/user1/file.txt user2@192.168.1.100:/home/user2/
	
## 从远程系统复制文件到本地系统

	scp [OPTION] [user@host:] [file] [file]		
	
例如，将远程系统中的 /home/user2/file.txt 文件复制到本地系统中的 /home/user1 目录下，可以使用如下命令：
	
	scp user2@192.168.1.100:/home/user2/file.txt /home/user1/
	
## 从远程系统复制目录到本地系统
	
	scp -r [user@host:] [directory] [directory]		
	
例如，将远程系统中的 /home/user2 目录复制到本地系统中的 /home/user1 目录下，可以使用如下命令：
	
	scp -r user2@192.168.1.100:/home/user2 /home/user1/
	
## 从本地系统复制目录到远程系统
	
	scp -r [directory] [user@host:] [directory]		
	
例如，将本地系统中的 /home/user1 目录复制到远程系统中的 /home/user2 目录下，可以使用如下命令：
	
	scp -r /home/user1 user2@192.168.1.100:/home/user2/
	
## 从远程系统复制文件到另一个远程系统
	
	scp [user@host:] [file] [user@host:] [file]		
	
例如，将远程系统中的 /home/user2/file.txt 文件复制到另一个远程系统中的 /home/user3 目录下，可以使用如下命令：
	
	scp user2@192.168.1.100:/home/user2/file.txt user3@192.168.1.101:/home/user3/
	
## 从本地系统复制文件到另一个远程系统
	
	scp [file] [user@host:] [file]				
	
例如，将本地系统中的 /home/user1/file.txt 文件复制到另一个远程系统中的 /home/user3 目录下，可以使用如下命令：
	
	scp /home/user1/file.txt user3@192.168.1.101:/home/user3/




```
-----
Copyright 2020 - 2025 @ [cheldon](https://github.com/cheldon-cn/).
