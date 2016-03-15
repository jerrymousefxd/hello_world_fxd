# hello_world_fxd
package com.filesupload;

import java.io.File;
import java.io.IOException;

import javax.servlet.http.HttpServletRequest;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.multipart.MultipartFile;

@Controller
public class FilesUploadAction {
	
	@RequestMapping("/login.do")
	/**
	 * 测试登陆跳转
	 * @return
	 */
	public String testLogin(){
		System.out.println("ssss");
		return "page/index";
	}
	@RequestMapping("/filesUpload.do")
	public String filesUpload(MultipartFile file,HttpServletRequest request){
		String path = request.getSession().getServletContext().getRealPath("files");
		System.out.println("存放文件路径:"+path);
		String fileName = file.getOriginalFilename();
		File targetFile = new File(path, fileName);
		if(!targetFile.exists()){  
            targetFile.mkdirs();  
        }
		try {
			file.transferTo(targetFile);
		} catch (IllegalStateException | IOException e) {
			e.printStackTrace();
		}
		
		String returnPath = request.getContextPath()+"/upload/"+fileName;
		request.setAttribute("filesUploadResult", returnPath);
		System.out.println("返回上传结果:"+returnPath);
		return "index/index";
	}
}

