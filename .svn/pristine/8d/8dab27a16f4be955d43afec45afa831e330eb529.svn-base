package com.xq.gymnasium.service.impl;


import java.util.ArrayList;
import java.util.Date;
import java.util.HashMap;
import java.util.HashSet;
import java.util.List;
import java.util.Map;
import java.util.Set;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.alibaba.fastjson.JSONArray;
import com.alibaba.fastjson.JSONException;
import com.alibaba.fastjson.JSONObject;
import com.xq.gymnasium.dao.GymnasiumMessageMapper;
import com.xq.gymnasium.dao.OrdersMapper;
import com.xq.gymnasium.dao.SiteMapper;
import com.xq.gymnasium.dao.SitetypeMapper;
import com.xq.gymnasium.model.GymnasiumMessage;
import com.xq.gymnasium.model.Orders;
import com.xq.gymnasium.model.Selectbyordersyd;
import com.xq.gymnasium.model.Site;
import com.xq.gymnasium.model.Sitetime;
import com.xq.gymnasium.model.Sitetype;
import com.xq.gymnasium.service.IOrdersService;
import com.xq.gymnasium.util.CodeUtil;
import com.xq.gymnasium.util.DateTools;

/**
 * 体育馆订单
 * @ClassName OrdersService
 * @Author yangweihang
 * @Date 2018年9月14日 上午9:34:30
 */
@Service("ios")
public class OrdersService implements IOrdersService {

	@Autowired
	private OrdersMapper om;
	
	@Autowired
	private SiteMapper ism;
	
	@Autowired
	private SitetypeMapper sm;
	
	@Autowired
	private GymnasiumMessageMapper imm;
	
	/**
	 * 批量录入订单
	 * yangweihang
	 * @Date 2018年9月14日 上午9:39:23
	 * @param sid	场地id
	 * @param oname	预订人
	 * @param ostarttime 预定开始时间
	 * @param oendtime	预定结束时间
	 * @return
	 */
	public String insertorders(Integer[] sid,String oname,String[] ostarttime,String[] oendtime,Integer gid) {
		if(oname == null) {
			return "login";
		}
		//查询当前的人是否有没有完成的订单
		List<Orders> os = om.selectByNotOrders(oname);
		if(os.size() >= 1) {
			//有预订订单未完成
			return "There are orders pending";
		}
		Double money = 0.0;
		DateTools dt = DateTools.getFactory();
		Date time = dt.formatDate(dt.formatDate(new Date(), "yyyy-MM-dd HH:ss:mm"), "yyyy-MM-dd HH:ss:mm");
		List<Orders> list = new ArrayList<Orders>();
		for (int i = 0; i < sid.length; i++) {
			//判断时间
			long stime = new Date().getTime();
			long sd = dt.formatDate(ostarttime[i], "yyyy-MM-dd HH:ss:mm").getTime();
			if(sd < stime) {
				return "99";
			}
			//按场地id查询场地
			Site s = ism.selectBySid(sid[i]);
			//按体育馆场地编号查询场地类别名称
			Sitetype st = sm.selectStnameBySnumber(s.getSnumber());
			Orders o = null;
			//如果是羽毛球场地的话就进行判断是否是按顺序预定的
			if(st.getStname().equals("羽毛球")) {
				//找到羽毛球场地对应的篮球场地是否预定了
				Integer selpstate = 0;
				for (int j = 0; j < 2; j++) {
					Map<String,Object> map = new HashMap<String,Object>();
					if(j == 0) {
						map.put("snumber", s.getSrsnumber()+"-A");
						map.put("gid", gid);
						System.out.println("map"+map);
						Site sels = ism.selectBySnumber(map);
						selpstate = selectreservationornot(sels.getSid(),ostarttime[i],oendtime[i],sels.getSnumber());
					}
					if(j == 1) {
						map.put("snumber", s.getSrsnumber()+"-B");
						map.put("gid", gid);
						Site sels = ism.selectBySnumber(map);
						selpstate = selectreservationornot(sels.getSid(),ostarttime[i],oendtime[i],sels.getSnumber());
					}
				}
				/*map.put("snumber", s.getSrsnumber());
				map.put("gid", gid);
				Site sels = ism.selectBySnumber(map);
				System.out.println("sels.getSnumber()"+sels);*/
				//int selpstate = selectreservationornot(sels.getSid(),ostarttime[i],oendtime[i],sels.getSnumber());
				if(selpstate == 1) {
					//羽毛球场地对应的篮球场地已经预定了
					return "1";
				}
				//获得是预定的第几块场地
				//String num = s.getSnumber().substring(s.getSnumber().length()-1, s.getSnumber().length());
				String code = s.getSnumber().split("-")[0];
				String num = s.getSnumber().split("-")[1];
				String oncode = oncode(code, num);
				/*String n = new Integer(num);*/
				//int number = n - 1;
				//String sitenumber = CodeUtil.sitenumber(2, number);
				if(num.equals("C")) {
					//查找上一块场地是否预定了
					int pstate = selectreservationornot(s.getSid(),ostarttime[i],oendtime[i],oncode);
					if(pstate == 1) {
						//抱歉，暂不可选，请选择其他场地
						return "s";
					}
				}
				//查找要预定的场地是否已经预定了
				int pstate1 = selectreservationornot(s.getSid(),ostarttime[i],oendtime[i],oncode);
				if(pstate1 == 1) {
					//羽毛球场地已经预定了
					return "2";
				}
			}
			if(st.getStname().equals("篮球")) {
				//找到篮球场地对应的羽毛球场地是否预定了
				List<Site> lists = ism.selectBySrsnumber(s.getSnumber());
				for (int j = 0; j < lists.size(); j++) {
					int pstate1 = selectreservationornot(lists.get(i).getSid(),ostarttime[i],oendtime[i],lists.get(i).getSnumber());
					if(pstate1 == 1) {
						//篮球场地对应的羽毛球场地已经预定了
						return "3";
					}
				}
				//获得是预定的第几块场地
				/*String num = s.getSnumber().substring(s.getSnumber().length()-1, s.getSnumber().length());
				Integer n = new Integer(num);
				String sitenumber = CodeUtil.sitenumber(1, n);*/
				/*System.out.println("snumber"+s.getSnumber());
				System.out.println("s.getSid()"+s.getSid());
				System.out.println("ostarttime[i]"+ostarttime[i]);
				System.out.println("oendtime[i]"+oendtime[i]);*/
				//查找要预定的场地是否已经预定了
				int pstate1 = selectreservationornot(s.getSid(),ostarttime[i],oendtime[i],s.getSnumber());
				if(pstate1 == 1) {
					//篮球场地已经预定不可预定
					return "4";
				}
			}
			money += s.getSprice();//总消费金额消费金额
			o = new Orders(null, sid[i], oname, dt.formatDate(ostarttime[i], "yyyy-MM-dd HH:ss:mm"), dt.formatDate(oendtime[i], "yyyy-MM-dd HH:ss:mm"), time, s.getSprice(), 1, null, null);
			list.add(o);
		}
		//批量录入订单
		int result = om.insertorders(list);
		if(result > 0) {
			return "insert.orders.success";
		}
		return "insert.orders.fail";
	}
	
	/**
	 * 获得场地编码
	 * yangweihang
	 * @Date 2018年9月25日 上午11:01:17
	 * @param code
	 * @param num
	 * @return
	 */
	public String oncode(String code,String num) {
		String oncode = "";
		if(num != "C") {
			if(num.equals("D")) {
				num = "C";
				oncode = code + "-" + num;
			}else if(num.equals("E")) {
				num = "D";
				oncode = code + "-" + num;
			}else if(num.equals("F")) {
				num = "E";
				oncode = code + "-" + num;
			}
		}else if(num.equals("C")) {
			oncode = code + "-" + num;
		}
		return oncode;
	}
	
	/**
	 * 查询该场地的指定时间是否已经预定了
	 * yangweihang
	 * @Date 2018年9月14日 上午10:11:53
	 * @param snumber 体育馆场地编号
	 * @return
	 */
	public int selectreservationornot(Integer sid,String ostarttime,String oendtime,String sitenumber) {
		int pstate = 0;
		//按场地编号查询这块场地是否预定了
		Map<String,Object> map = new HashMap<String,Object>();
		map.put("sid", sid);
		map.put("ostarttime", ostarttime);
		map.put("oendtime", oendtime);
		map.put("snumber", sitenumber);
		System.out.println("sitenumber"+sitenumber);
		System.out.println("map"+map);
		List<Site> s = ism.selectnotororder(map);
		if(s.size() >= 1) {
			pstate = 1;//已经预定了
		}
		return pstate;
	}
	
	/**
	 * 查询该人的订单
	 * yangweihang
	 * @Date 2018年9月14日 下午4:29:17
	 * @param oname	预订人
	 * @param starttime	开始时间
	 * @param endtime	结束时间
	 * @return
	 */
	public List<Map<String,Object>> selectByOname(String oname,String starttime,String endtime) {
		DateTools dt = DateTools.getFactory();
		Map<String,Object> map = new HashMap<String,Object>();
		map.put("oname", oname);
		map.put("starttime", starttime);
		map.put("endtime", endtime);
		List<Map<String,Object>> list = om.selectByOname(map);
		StringBuffer sb = new StringBuffer();
		for (Map<String,Object> m : list) {
			//替换时间格式
			String start = dt.formatDate((Date)m.get("ostarttime"), "yyyy.MM.dd HH:ss");
			String end = dt.formatDate((Date)m.get("oendtime"), "HH:ss");
			sb.append(start);
			sb.append("-");
			sb.append(end);
			m.replace("ostarttime", sb.toString());
			//状态
			if((int)m.get("state") == 1) {
				m.replace("state", "已预订");
				//判断是否是使用过了
				long date = dt.getMilliseconds((Date)m.get("oendtime"));
				Date today = new Date();
				long time = today.getTime();
				if(date < time) {
					m.replace("state", "已使用");
				}
			}else if((int)m.get("state") == 2) {
				m.replace("state", "已退订");
			}
			//清空StringBuffer
			sb.setLength(0);
		}
		return list;
	}
	
	/**
	 * 批量退订
	 * yangweihang
	 * @Date 2018年9月14日 下午8:08:39
	 * @param oid 订单id
	 * @return
	 */
	public String updatestate(Integer[] oid) {
		List<Orders> list = new ArrayList<Orders>();
		for (int i = 0; i < oid.length; i++) {
			//查询是否存在不是已预订的状态
			Orders o = om.selectbystate(oid[i]);
			if(o != null) {
				if(o.getState() == 1) {
					//您好，此预订不能退订，谢谢
					return "Hello, this reservation is unrefundable, thank you";
				}else if(o.getState() == 2) {
					//您已退订，请勿重复操作
					return "Please do not repeat after you unsubscribe";
				}
			}
			Orders os = new Orders();
			os.setOid(oid[i]);
			list.add(os);
		}
		int result = om.updatestate(list);
		if(result > 0) {
			return "order.unsubscribe.success";
		}
		return "order.unsubscribe.fail";
	}
	
	/**
	 * 查询场地预定信息
	 * yangweihang
	 * @Date 2018年9月15日 上午11:09:25
	 * @param map
	 * @return
	 */
	public List<Map<String,Object>> selectbyorders(String starttime,String endtime,Integer pageNum,Integer pageSize){
		endtime = endtime +" 23:59:59";
		DateTools dt = DateTools.getFactory();
		Map<String,Object> map = new HashMap<String,Object>();
		map.put("starttime", starttime);
		map.put("endtime", endtime);
		map.put("pageNum", pageNum);
		map.put("pageSize", pageSize);
		List<Map<String, Object>> list = om.selectbyorders(map);
		for (Map<String, Object> m : list) {
			StringBuffer sb = new StringBuffer();
			//替换时间格式
			Date d = (Date)m.get("ostarttime");
			String start = dt.formatDate(d, "yyyy.MM.dd HH:ss");
			String end = dt.formatDate((Date)m.get("oendtime"), "HH:ss");
			sb.append(start);
			sb.append("-");
			sb.append(end);
			m.replace("ostarttime", sb.toString());
			String otime = dt.formatDate((Date)m.get("otime"), "yyyy-MM-dd HH:ss:mm");
			m.replace("otime", otime);
			//判断开始时间有没有超出当前时间
			if(d.getTime() > new Date().getTime()) {
				//超时了
				m.put("state", 1);
			}else {
				m.put("state", 2);
			}
		}
		return list;
	}
	
	/**
	 * 查询场地预定信息总数
	 * yangweihang
	 * @Date 2018年9月20日 下午2:06:00
	 * @param map
	 * @return
	 */
	public List<Map<String,Object>> selectbyorderscount(String starttime,String endtime,Integer pageNum,Integer pageSize){
		Map<String,Object> map = new HashMap<String,Object>();
		map.put("starttime", starttime);
		map.put("endtime", endtime);
		map.put("pageNum", pageNum);
		map.put("pageSize", pageSize);
		List<Map<String, Object>> list = om.selectbyorderscount(map);
		return list;
	}
	
	/**
	 * 按日期查询场馆预定情况
	 * yangweihang
	 * @Date 2018年9月15日 上午11:09:25
	 * @param map
	 * @return
	 */
	public List<Map<String,Object>> selectbyordersyd(Selectbyordersyd sb){
		DateTools dt = DateTools.getFactory();
		List<Map<String, Object>> list = om.selectbyordersyd(sb);
		for (Map<String, Object> m : list) {
			m.replace("ostarttime", dt.formatDate((Date)m.get("ostarttime"), "yyyy-MM-dd HH:ss:mm"));
			m.replace("oendtime", dt.formatDate((Date)m.get("oendtime"), "yyyy-MM-dd HH:ss:mm"));
		}
		return list;
	}
	
	/**
	 * 按日期查询场馆预定情况
	 * yangweihang
	 * @Date 2018年9月15日 上午11:09:25
	 * @param map
	 * @return
	 * @throws JSONException 
	 */
	public JSONArray selectbyordersyds(Selectbyordersyd sb) throws Exception{
		JSONArray array = null;//大
		JSONObject jsonObject = null;
		Set<String> set = null;
		String f = "";
		DateTools dt = DateTools.getFactory();
		array = new JSONArray();
		for (int l = 0; l < 7; l++) {
			if(l != 0) {
				Date d = dt.formatDate(sb.getServenday(), "yyyy-MM-dd HH:ss:mm");
				Date d1 = dt.addDay(d, 1);
				f = dt.formatDate(d1, "yyyy-MM-dd HH:ss:mm");
				sb.setServenday(f);
			}
			List<Map<String, Object>> list = om.selectbyordersyd(sb);
			if(!list.isEmpty() && list != null) {
					String one1 = dt.formatDate((Date) list.get(0).get("ostarttime"), "yyyy-MM-dd HH:ss:mm").split(" ")[0];
					String two1 = one1.split("-")[1];
					String three1 = one1.split("-")[2];
					String str = two1 + "." + three1;
					jsonObject = new JSONObject();
					set = new HashSet<String>();
					jsonObject.put("time", str);
					for (Map<String, Object> m : list) {
						String one = dt.formatDate((Date) m.get("ostarttime"), "yyyy-MM-dd HH:ss:mm").split(" ")[0];
						String o = dt.formatDate((Date) m.get("ostarttime"), "yyyy-MM-dd HH:ss:mm").split(" ")[1];
						String two = one.split("-")[1];
						String three = one.split("-")[2];
						String time = two + "." + three;
						if (jsonObject.get("time").equals(time)) {
							set.add(o);
						}
					}
					jsonObject.put("value", set);
					array.add(jsonObject);
			}else {
				if(f != null && !f.equals("")) {
					jsonObject = new JSONObject();
					String one = f.split(" ")[0];
					String two = one.split("-")[1];
					String three = one.split("-")[2];
					String time = two + "." + three;
					jsonObject.put("time", time);
					jsonObject.put("value", new String[] {});
					array.add(jsonObject);
				}else {
					jsonObject = new JSONObject();
					String one = sb.getServenday().split(" ")[0];
					String two = one.split("-")[1];
					String three = one.split("-")[2];
					String time = two + "." + three;
					jsonObject.put("time", time);
					jsonObject.put("value",  new String[] {});
					array.add(jsonObject);
				}
			}
		}
		System.out.println("array"+array);
		return array;
	}
	
	/**
	 * 按日期查询场馆预定情况前台
	 * yangweihang
	 * @Date 2018年9月15日 上午11:09:25
	 * @param map
	 * @return
	 * @throws JSONException 
	 */
	public List<Sitetime> selectbyordersydss(Selectbyordersyd sb){
		DateTools dt = DateTools.getFactory();
		List<Map<String, Object>> list = om.selectbyordersydss(sb);
		List<String> l = new ArrayList<String>();
		List<Integer> sid = new ArrayList<Integer>();
		List<String> time = null;
		List<Sitetime> st = new ArrayList<Sitetime>();
		for (Map<String, Object> m : list) {
			if(!sid.contains((Integer)m.get("sid"))) {
				sid.add((Integer) m.get("sid"));
			}
		}
		for (int i = 0; i < sid.size(); i++) {
			Sitetime s = new Sitetime();
			s.setSid(sid.get(i));
			st.add(s);
		}
		Integer sids = 0;
		for (int i = 0; i < sid.size(); i++) {
			time = new ArrayList<String>();
			for(Map<String, Object> m : list) {
				if(sid.get(i) == (Integer) m.get("sid")) {
					if(m.get("ostarttime") != null) {
						String o = dt.formatDate((Date) m.get("ostarttime"), "yyyy-MM-dd HH:ss:mm").split(" ")[1];
						time.add(o);
						sids = (Integer) m.get("sid");
					}
				}
			}
			for (int k = 0; k < st.size(); k++) {
				if(st.get(k).getSid().equals(sids)) {
					st.get(k).setTime(time);
					break;
				}
			}
		}
		return st;
	}
	
	/**
	 * 判断体育馆休息日期
	 * yangweihang
	 * @Date 2018年9月26日 下午1:31:51
	 * @param sb
	 * @return
	 */
	public String selectbyweek(Integer gid,String servenday) {
		GymnasiumMessage g = imm.selectbygid(gid);
		String str = "1";
		if(g != null) {
			//场馆闭馆星期
			String[] time = g.getGclosedtime().split(",");
			int week = 0;
			try {
				week = DateTools.dayForWeek(servenday);
			} catch (Exception e1) {
				e1.printStackTrace();
			}
			for (int i = 0; i < time.length; i++) {
				if(week == new Integer(time[i])) {
					str = "0";
					return str;
				}
			}
			//场馆闭馆状态
			if(g.getState() == 2) {
				str = "0";
			}
		}
		return str;
		/*System.out.println("sb"+sb);
		DateTools dt = DateTools.getFactory();
		List<Map<String, Object>> list = om.selectbyordersydss(sb);
		System.out.println("list"+list);
		int a = 0;
		boolean flag = true;
		String str = "1";
		//判断星期几
		for (Map<String, Object> m : list) {
			String loctime = (String)m.get("gclosedtime");
			String[] time = loctime.split(",");
			if(time != null) {
				int week = 0;
				try {
					week = DateTools.dayForWeek(sb.getServenday());
				} catch (Exception e) {
					e.printStackTrace();
				}
				for (int i = 0; i < time.length; i++) {
					if(week == new Integer(time[i])) {
						flag = false;
						str = "0";
						break;
					}
				}
			}
			a++;
		}
		return str;*/
	}
	
	/**
	 * 查询当天的使用的人数和场地数
	 * yangweihang
	 * @Date 2018年9月17日 下午1:37:54
	 * @return
	 */
	public Map<String,Object> selectbycount(){
		Map<String, Object> map = om.selectbycount();
		return map;
	}
}
