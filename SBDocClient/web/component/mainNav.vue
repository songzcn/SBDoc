<template>
    <el-row id="navBar" class="row" :style="transparent?{height:'60px','backgroundColor':'rgba(0,0,0,0.3)',left:0,top:0,position:'absolute'}:{height:'60px','backgroundColor':'white'}">
        <slot name="other"></slot>
        <el-col class="col" :span="3" style="text-align: left;line-height: 60px;color:#20A0FF ;font-size: 30px;padding-left: 20px">
            <a style="text-decoration: none;cursor: pointer;color: inherit" href="/">SBDoc</a>
        </el-col>
        <el-col class="col" :span="2" style="text-align: center;line-height: 60px;">
            <slot name="slot3">
            </slot>
        </el-col>
        <el-col class="col" :span="2" style="text-align: center;line-height: 60px;">
            <slot name="slot4">
            </slot>
        </el-col>
        <el-col class="col" :span="1" style="text-align: center;line-height: 60px;">

        </el-col>
        <el-col class="col" :span="8" style="text-align: center;line-height: 60px;font-size: 25px;color: #20A0FF">
            <slot name="title">
            </slot>
        </el-col>
        <el-col class="col" :span="2" style="text-align: center;line-height: 60px;">
            <slot name="slot1">
            </slot>
        </el-col>
        <el-col class="col" :span="2" style="text-align: center;line-height: 60px;">
            <slot name="slot2">
            </slot>
        </el-col>
        <el-col class="col" :span="4" v-if="isLogin" style="white-space: nowrap;text-align: center;line-height: 60px">
            <img v-proxy="img" style="width: 40px;height: 40px; border-radius:50%;margin-top: 10px">&nbsp;
            <el-dropdown @command="handleCommand" style="top: -15px;">
                <span class="el-dropdown-link" style="color: #20A0FF;cursor: pointer">
                    <el-badge is-dot class="msgBadge" v-if="newMsg">
                        {{name}}
                    </el-badge>
                    <span v-else>
                        {{name}}
                    </span>
                    <i class="el-icon-caret-bottom el-icon--right"></i>
                </span>
                <el-dropdown-menu slot="dropdown">
                    <el-dropdown-item command="team" v-if="team">团队首页</el-dropdown-item>
                    <el-dropdown-item command="list">返回列表</el-dropdown-item>
                    <el-dropdown-item command="apply">团队申请</el-dropdown-item>
                    <el-dropdown-item command="setting">个人设置</el-dropdown-item>
                    <el-dropdown-item command="message">
                        <el-badge is-dot class="msgBadge" v-if="newMsg">
                            消息中心
                        </el-badge>
                        <span v-else>
                            消息中心
                        </span>
                    </el-dropdown-item>
                    <el-dropdown-item>
                        Proxy:<br>
                        <el-switch v-model="proxy" on-color="#13ce66" off-color="#ff4949" @click.native.stop="">
                        </el-switch>
                    </el-dropdown-item>
                    <el-dropdown-item command="update">检查更新</el-dropdown-item>
                    <el-dropdown-item command="quit">退出</el-dropdown-item>
                </el-dropdown-menu>
            </el-dropdown>
        </el-col>
        <el-col class="col" :span="2" style="text-align: center;line-height: 60px" v-if="!isLogin">
            <el-button type="info" onclick="location='/html/web/login/login.html'">登录</el-button>
        </el-col>
        <el-col class="col" :span="2" style="text-align: center;line-height: 60px" v-if="!isLogin">
            <el-button type="success"  onclick="location='/html/web/register/register.html'">注册</el-button>
        </el-col>
        <el-dialog title="团队申请" v-model="showTeam" size="small" ref="team">
            <el-form ref="form" label-width="100px">
                <el-form-item label="团队ID">
                    <el-input  style="width: 80%"  v-model="applyName" placeholder="请输入你要申请的团队ID"></el-input>
                </el-form-item>
                <el-form-item label="备注">
                    <el-input type="textarea" :rows="2"  style="width: 80%"  v-model="applyDis" placeholder="请输入你要申请的备注"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
            <el-button @click="showTeam = false">取 消</el-button>
            <el-button type="primary" @click="applyTeam" :loading="applyPending">确 定</el-button>
        </span>
        </el-dialog>
    </el-row>
</template>
<script>
    var con=require("../../../config.json");
    var proxyImg=require("../director/proxyImg")
    module.exports={
        props:["transparent"],
        data:function () {
            return {
                isLogin:session.get('id')?true:false,
                img:session.get('photo'),
                name:session.get("name"),
                team:session.get("teamId"),
                showTeam:false,
                applyPending:false,
                applyName:"",
                applyDis:"",
                newMsg:false,
                proxy:session.get("proxy")?true:false
            }
        },
        directives:{
            proxy:proxyImg
        },
        watch:{
            proxy:function (val) {
                if(val)
                {
                    session.set("proxy",1);
                    $.tip("Proxy代理已开启",1)
                }
                else
                {
                    session.remove("proxy");
                    $.tip("Proxy代理已关闭",1)
                }
            }
        },
        methods:{
            handleCommand:function (command) {
                if(command=="team")
                {
                    location.href="/html/web/team/team.html"
                }
                else if(command=="list")
                {
                    location.href="/html/web/project/project.html"
                }
                else if(command=="apply")
                {
                    this.showTeam=true;
                    document.getElementById("navBar").style.zIndex="";
                    this.$refs.team.$on("close",function () {
                        document.getElementById("navBar").style.zIndex=100;
                    })
                }
                else if(command=="setting")
                {
                    location.href="/html/web/person/person.html"
                }
                else if(command=="message")
                {
                    var _this=this;
                    $.startHud();
                    net.get("/message/list",{
                        page:0
                    }).then(function (data) {
                        $.stopHud();
                        if(data.code==200)
                        {
                            _this.newMsg=false;
                            document.getElementById("navBar").style.zIndex="";
                            var child=$.showBox(_this,"message",{
                                propArr:data.data
                            });
                            child.$on("close",function () {
                                document.getElementById("navBar").style.zIndex=100;
                            })
                        }
                        else
                        {
                            $.notify(data.msg,0)
                        }
                    })
                }
                else if(command=="update")
                {
                    var xml=new XMLHttpRequest();
                    $.startHud();
                    xml.onreadystatechange=function () {
                        if(xml.readyState==4 && xml.status==200)
                        {
                            $.stopHud();
                            var obj=JSON.parse(xml.responseText);
                            var verArr=obj[0].name.split(".");
                            var verLocalArr=con.version.split(".");
                            var bNew=false;
                            for(var i=0;i<3;i++)
                            {
                                if(verArr[i]>verLocalArr[i])
                                {
                                    bNew=true;
                                    break;
                                }
                                else if(verArr[i]<verLocalArr[i])
                                {
                                    break;
                                }
                            }
                            if(bNew)
                            {
                                $.confirm("已发现新版本"+verArr.join(".")+" 是否现在下载？",function () {
                                    window.open(obj[0].zipball_url,"_blank");
                                })
                            }
                            else
                            {
                                $.tip("已经是最新版本了",1);
                            }
                        }
                    }
                    xml.open("GET","https://api.github.com/repos/sx1989827/SBDoc/tags?timestamp="+(new Date()).getTime(),true);
                    xml.send();
                }
                else if(command=="quit")
                {
                    var _this=this;
                    net.post("/user/logout",{},{
                        "content-type":"application/x-www-form-urlencoded"
                    }).then(function (data) {
                        if(data.code==200)
                        {
                            _this.$notify({
                                title: '退出成功',
                                type: 'success'
                            });
                            session.clear();
                            setTimeout(function () {
                                location.href="/";
                            },1000)

                        }
                    })
                }
            },
            applyTeam:function () {
                if(!this.applyName)
                {
                    $.tip("请输入团队ID",0);
                    return;
                }
                this.applyPending=true;
                var _this=this;
                net.put("/team/userapply",{
                    id:this.applyName,
                    dis:this.applyDis
                }).then(function (data) {
                    _this.applyPending=false;
                    _this.applyName="";
                    _this.applyDis=""
                    if(data.code==200)
                    {
                        $.notify("请求已发送，等待团队管理员响应",1);
                        _this.showTeam=false;
                    }
                    else
                    {
                        $.notify(data.msg,0);
                    }
                })
            }
        },
        created:function () {
            var ele;
            this.$nextTick(function () {
                ele=document.getElementById("navBar");
                ele.style.zIndex=100;
            })
            var _this=this;
            if(this.transparent)
            {
                $.addEventListener(window,"scroll",function () {
                    if(document.body.scrollTop>60)
                    {
                        ele.style.position="fixed";
                        ele.style.top=0;
                        ele.style.backgroundColor="rgb(39,52,68)"
                    }
                    else
                    {
                        ele.style.top=0;
                        ele.style.backgroundColor="rgba(0,0,0,0.3)"
                        ele.style.position="absolute";
                    }
                })
            }
            this.$parent.$on("updatePhoto",function (val) {
                _this.img=val;
            })
            if(session.get("id"))
            {
                net.get("/message/new").then(function (data) {
                    if(data.code==200)
                    {
                        _this.newMsg=data.data;
                    }
                    else
                    {
                        $.notify(data.msg,0)
                    }
                })
            }
        }
    }
</script>
