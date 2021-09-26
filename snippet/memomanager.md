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

# 18 Parse Segments 

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
  ```

# 19 QT_NO_CLIPBOARD

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
2. but here we use another way in case that we have no so must information about the clipboard jni method;
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
	in qtgui-config_p.h or qtgui-config.h, we can find any macro defination;

3. we edit the file 'qtgui-config_p.h' by appending the defination of the macro  QT_NO_CLIPBOARD ;
   then compile the source code again and test the code;
   OK,we success; prove that that method is OK;

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

	so what create the file qtgui-config.pri? How to make clipboard disabled ?

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
   


# 20 debug with add-dsym in AS

## detail version

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


## short version

1.Debug ---> app --->LLDB
2.pause
3.add-dsym 
4.resume
5.debug

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

# 21 init Qt resource

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



-----
Copyright 2020 - 2021 @ [cheldon](https://github.com/cheldon-cn/).
