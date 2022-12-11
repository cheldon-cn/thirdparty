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


#define MEMBLOCKSIZE	(1024*1024*10)	//10M space
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
		m_buf = new char[MEMBLOCKSIZE];
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
	if (MEMBLOCKSIZE < size + m_pos)
	{
		m_idx++;
		if (m_idx >= m_list.Size())
		{
			m_buf = new char[((MEMBLOCKSIZE > size) ? MEMBLOCKSIZE : size)];
			m_list.Add(m_buf);
		}
		else if (MEMBLOCKSIZE >= size)
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

## 1.STEPS
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
## 2. INSTANCE
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

# 75 .build cario on windows

## 1.STEPS
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
## 2. INSTANCE
```
Administrator@YR170812-LMCY /cygdrive/f/cario/cairo-1.16.0/bd
$  ../configure --prefix=/cygdrive/f/cario/install/cario --host=x86_64-w64-mingw32  png_CFLAGS='-I/cygdrive/f/cario/install/png/include' png_LIBS='-L/cygdrive/f/cario/install/png/lib  -lpng16' png_REQUIRES="libpng16" pixman_CFLAGS='-I/cygdrive/f/cario/install/pixman/include/pixman-1' pixman_LIBS='-L/cygdrive/f/cario/install/pixman/lib  -lpixman-1'  CFLAGS='-I/cygdrive/f/cario/install/zlib/include' LIBS='-L/cygdrive/f/cario/install/zlib/lib -lz' --enable-ft=no

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
MFCOMMONEXPORT gisLONG API_FileDirPath(CString strDir, CString strFile, CString& strDirPath)
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
	gisLONG nIndex = 0;

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

-----
Copyright 2020 - 2022 @ [cheldon](https://github.com/cheldon-cn/).
