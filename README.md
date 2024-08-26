import React, {useState} from "react";
import {Box, Button, TextField, Typography} from "@mui/material";
import useMediaQuery from "@mui/material/useMediaQuery";
import {useTranslation} from "react-i18next";
import useStyles1 from "../../assets/style/style";
import useStyles2 from "../../assets/style/style2";
import Header from "../../components/Header";
import AddMenuItem from "../../components/addMenu/AddMenuItem";
import MediaPlayer from "../../components/MediaPlayer";
import {useTheme} from "@mui/material/styles";
import {tokens} from "../../theme";
import axios from "axios";
import Cookie from "js-cookie";
import {jwtDecode} from "jwt-decode";
import AddMenuItemTest from "../../components/addMenu/AddMenuItemTest";

const Menu = () => {
        const theme1 = useTheme();
        const colors = tokens(theme1.palette.mode);
        const isMaxWidth1280 = useMediaQuery("(max-width:1280px)");
        const styles1 = useStyles1();
        const styles2 = useStyles2();
        const style = isMaxWidth1280 ? styles2 : styles1;
        const {t} = useTranslation();
        const [dialogOpen, setDialogOpen] = useState(false);
        const [selectedId, setSelectedId] = useState(null);
        const [error, setError] = useState("");
        const [menuName, setMenuName] = useState("");
        const [nameError, setNameError] = useState(false);
        const MAX_FILE_SIZE_MB = 5;
        const cookieToken = Cookie.get("token");
        const decodedToken = jwtDecode(cookieToken);
        const usrid = decodedToken.user_id;
        console.log(usrid);
        const [files, setFiles] = useState({
            main_greeting: null,
            main_goodbye: null,
            main_invalid: null,
            1: null,
            2: null,
            3: null,
            4: null,
            5: null,
            6: null,
            7: null,
            8: null,
            9: null,
            0: null,
        });

        const openDialog = (id) => {
            setSelectedId(id);
            setDialogOpen(true);
        };

        const closeDialog = () => {
            setDialogOpen(false);
            setSelectedId(null);
        };

        const handleFileChange = (e) => {
            const {name, files} = e.target;
            const file = files[0];
            const fileSizeMB = (file.size / (1024 * 1024)).toFixed(2);

            if (file.size > MAX_FILE_SIZE_MB * 1024 * 1024) {
                const errorMessage = ${t('max_size_limit_1')}  ${MAX_FILE_SIZE_MB} ${t('max_size_limit_2')} ${t('max_size_limit_3')} ${fileSizeMB} MB.;
                setError(errorMessage);
                return;
            } else {
                setError('');
            }
            setFiles(prev => ({
                ...prev,
                [name]: file
            }));
            handleFileDataChange(name, file);
        };


        const handleFileDataChange = (id, file) => {
            setFiles(prev => ({
                ...prev,
                [id]: file
            }));
        };
        const handleSave = () => {
            if (!menuName || menuName.trim() === "") {
                setNameError(true);
            } else {
                setNameError(false);
                const token = Cookie.get("token");
                const endpoint = 'http://192.168.1.72:8081/api/v1/save-menu';
                const formData = new FormData();
                formData.append('user_id', usrid);
                formData.append('menu_name', menuName);
                Object.keys(files).forEach((key) => {
                    if (files[key] && key !== 'user_id') {
                        formData.append(key, files[key]);
                    }
                });
                axios.post(endpoint, formData, {
                    headers: {
                        'Authorization': Bearer ${token},
                        'Content-Type': 'multipart/form-data'
                    }
                })
                    .then(response => {
                        console.log('Success:', response.data);
                        console.log(files);
                    })
                    .catch(error => {
                        console.error('Error:', error);
                        console.log('Error log:', formData);
                    });
            }
        };

        const handleResetFiles = () => {
            if (selectedId) {
                const updatedFiles = {...files};
                Object.keys(updatedFiles).forEach((key) => {
                    if (key.startsWith(${selectedId}_) && key.includes('_sub_')) {
                        delete updatedFiles[key];
                    }
                });

                setFiles(updatedFiles);
            } else {
                setFiles({});
            }
            setError('');
        };

        return (
            <Box m="20px">
                <Box sx={{display: "flex", justifyContent: "space-between", gap: "20px"}}>
                    <Box sx={{width: "100%"}}>
                        <Box>
                            <Box>
                                <Box
                                    sx={style.mainMenu.firstBox}>
                                    <Header subtitle="Menu Creating"/>
                                    <TextField
                                        label={t('menu_name')} // Label for the text field
                                        value={menuName}
                                        onChange={(e) => setMenuName(e.target.value)}
                                        error={nameError} // Error state for validation
                                        helperText={nameError ? t('required') : ''}
                                        sx={{marginBottom: 2}}
                                    />
                                    <MediaPlayer files={files}/>
                                </Box>
                                <Box>
                                    <Box
                                        sx={style.mainMenu.secondBox}>
                                        <Box
                                            sx={style.mainMenu.thirdBox}>
                                            <Box
                                                sx={style.mainMenu.fourthBox}>
                                                <Typography variant={"h4"} sx={{fontWeight: "700"}}>Greeting</Typography>
                                                <Box
                                                    sx={style.mainMenu.headInput}>
                                                    <input id="main_greeting-file" name="main_greeting" type="file"
                                                           accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                                           style={style.mainMenu.inputHidden}
                                                           onChange={handleFileChange}/>
                                                    <label htmlFor="main_greeting-file" style={{cursor: "pointer"}}>Choose
                                                        File</label>
                                                </Box>
                                            </Box>
                                            <Box sx={{display: "flex", alignItems: "center"}}>
                                                <Typography variant={"h4"} sx={{fontWeight: "700"}}>Goodbye</Typography>
                                                <Box
                                                    sx={style.mainMenu.headInput}>
                                                    <input id="main_goodbye-file" name="main_goodbye" type="file"
                                                           accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                                           style={style.mainMenu.inputHidden}
                                                           onChange={handleFileChange}/>
                                                    <label htmlFor="main_goodbye-file" style={{cursor: "pointer"}}>Choose
                                                        File</label>
                                                </Box>
                                            </Box>
                                            <Box sx={{display: "flex", alignItems: "center"}}>
                                                <Typography variant={"h4"} sx={{fontWeight: "700"}}>Invalid</Typography>
                                                <Box
                                                    sx={style.mainMenu.headInput}>
                                                    <input id="main_invalid-file" name="main_invalid" type="file"
                                                           accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                                           style={style.mainMenu.inputHidden}
                                                           onChange={handleFileChange}/>
                                                    <label htmlFor="main_invalid-file" style={{cursor: "pointer"}}>Choose
                                                        File</label>
                                                </Box>
                                            </Box>
                                        </Box>
                                        <Box sx={style.mainMenu.fifthBox}>
                                            {[1, 2, 3].map((number) => (
                                                <Box
                                                    sx={style.mainMenu.bodyInput}
                                                    key={number}>
                                                    <Button
                                                        id={${number}-file}
                                                        name={${number}}
                                                        style={style.mainMenu.buttonFonSize}
                                                        onClick={() => openDialog(${number})}
                                                    >
                                                        {number}
                                                    </Button>
                                                </Box>
                                            ))}
                                        </Box>
                                        <Box sx={style.mainMenu.sixthBox}>
                                            {[4, 5, 6].map((number) => (
                                                <Box
                                                    sx={style.mainMenu.bodyInput}
                                                    key={number}>
                                                    <Button
                                                        id={${number}-file}
                                                        name={${number}}
                                                        style={style.mainMenu.buttonFonSize}
                                                        onClick={() => openDialog(${number})}
                                                    >
                                                        {number}
                                                    </Button>
                                                </Box>
                                            ))}
                                        </Box>
                                        <Box
                                            sx={style.mainMenu.seventhBox}>
                                            {[7, 8, 9].map((number) => (
                                                <Box
                                                    sx={style.mainMenu.bodyInput}
                                                    key={number}>
                                                    <Button
                                                        id={${number}-file}
                                                        name={${number}}
                                                        style={style.mainMenu.buttonFonSize}
                                                        onClick={() => openDialog(${number})}
                                                    >
                                                        {number}
                                                    </Button>
                                                </Box>
                                            ))}
                                        </Box>
                                        <Box sx={style.mainMenu.eighthBox}>
                                            <Box
                                                sx={style.mainMenu.bodyInput}>
                                                <Button
                                                    id={0-file}
                                                    name={0}
                                                    style={style.mainMenu.buttonFonSize}
                                                    onClick={() => openDialog(0)}
                                                >
                                                    0
                                                </Button>
                                            </Box>
                                        </Box>
                                        <Box sx={{color: colors.redAccent[500], mt: "15px"}}>
                                            {error && <Typography variant="body2" sx={{
                                                fontSize: "18px",
                                                fontWeight: "700",
                                            }}>{error}</Typography>}
                                        </Box>
                                        <Box
                                            sx={style.mainMenu.saveButtonBox}>

                                            <Button
                                                sx={style.mainMenu.saveButton}
                                                onClick={() => {
                                                    handleSave()
                                                }}>{t('save')}</Button>
                                        </Box>
                                    </Box>
                                </Box>
                            </Box>
                        </Box>
                    </Box>
                </Box>
                <AddMenuItemTest
                    open={dialogOpen}
                    handleClose={closeDialog}
                    selectedId={selectedId}
                    files={files}
                    handleFileDataChange={handleFileDataChange}
                    onReset={handleResetFiles}
                />
            </Box>
        );
    }
;

export default Menu;
buradan bir sub menu acilsin ve oda bu olsun 
import React, { useState, useRef } from 'react';
import { Box, Button, Dialog, DialogContent, DialogTitle, Typography } from '@mui/material';
import CloseIcon from '@mui/icons-material/Close';
import { useTranslation } from "react-i18next";
import { tokens } from "../../theme";
import useMediaQuery from "@mui/material/useMediaQuery";
import { useTheme } from "@mui/material/styles";
import useStyles1 from "../../assets/style/style";
import useStyles2 from "../../assets/style/style2";
import AddMenuItem from "./AddMenuItem";

const AddMenuItemTest = ({ open, handleClose, selectedId, handleFileDataChange, onReset }) => {
    const theme1 = useTheme();
    const colors = tokens(theme1.palette.mode);
    const { t } = useTranslation();
    const fullScreen = useMediaQuery(theme1.breakpoints.down('md'));
    const [isSubmenuPage, setIsSubmenuPage] = useState(true);
    const [error, setError] = useState('');
    const fileInputRefs = useRef({});
    const MAX_FILE_SIZE_MB = 5;
    const isMaxWidth1280 = useMediaQuery("(max-width:1280px)");
    const styles1 = useStyles1();
    const styles2 = useStyles2();
    const style = isMaxWidth1280 ? styles2 : styles1;
    const [localSelectedId, setLocalSelectedId] = useState(null);
    const [files, setFiles] = useState({
        [${localSelectedId}_sub_main_greeting]: null,
        [${localSelectedId}_sub_main_goodbye]: null,
        [${localSelectedId}_sub_main_invalid]: null,
        [${localSelectedId}_sub_1]: null,
        [${localSelectedId}_sub_2]: null,
        [${localSelectedId}_sub_3]: null,
        [${localSelectedId}_sub_4]: null,
        [${localSelectedId}_sub_5]: null,
        [${localSelectedId}_sub_6]: null,
        [${localSelectedId}_sub_7]: null,
        [${localSelectedId}_sub_8]: null,
        [${localSelectedId}_sub_9]: null,
        [${localSelectedId}_sub_0]: null,
    });

    const handleFileChange = (e) => {
        const { name, files } = e.target;
        const file = files[0];
        const fileSizeMB = (file.size / (1024 * 1024)).toFixed(2);
        if (file.size > MAX_FILE_SIZE_MB * 1024 * 1024) {
            const errorMessage = ${t('max_size_limit_1')} ${MAX_FILE_SIZE_MB} ${t('max_size_limit_2')} ${t('max_size_limit_3')} ${fileSizeMB} MB.;
            setError(errorMessage);
        } else {
            setError('');
            setFiles(prevFiles => ({
                ...prevFiles,
                [name]: file
            }));
            handleFileDataChange(name, file);
        }
    };

    const handleRefresh = () => {
        Object.values(fileInputRefs.current).forEach(input => {
            if (input) input.value = '';
        });
        setFiles({
            [${localSelectedId}_sub_main_greeting]: null,
            [${localSelectedId}_sub_main_goodbye]: null,
            [${localSelectedId}_sub_main_invalid]: null,
            [${localSelectedId}_sub_1]: null,
            [${localSelectedId}_sub_2]: null,
            [${localSelectedId}_sub_3]: null,
            [${localSelectedId}_sub_4]: null,
            [${localSelectedId}_sub_5]: null,
            [${localSelectedId}_sub_6]: null,
            [${localSelectedId}_sub_7]: null,
            [${localSelectedId}_sub_8]: null,
            [${localSelectedId}_sub_9]: null,
            [${localSelectedId}_sub_0]: null,
        });
        setError('');
        if (onReset) {
            onReset();
        }
    };

    const [dialogOpen, setDialogOpen] = useState(false);

    const openDialog = (id) => {
        setLocalSelectedId(id);
        setDialogOpen(true);
    };

    const closeDialog = () => {
        setDialogOpen(false);
        setLocalSelectedId(null);
    };

    return (
        <Box>
            <Dialog
                fullScreen={fullScreen}
                open={open}
                onClose={handleClose}
                aria-labelledby="responsive-dialog-title"
                PaperProps={{
                    sx: {
                        maxWidth: '800px',
                        width: '800px',
                        maxHeight: "750px",
                        height: isMaxWidth1280 ? '550px' : "750px",
                    },
                }}
            >
                <Box sx={{
                    display: 'flex',
                    justifyContent: 'space-between',
                    backgroundColor: colors.addDialog[100],
                    color: colors.addDialog[400]
                }}>
                    <DialogTitle id="responsive-dialog-title"
                                 sx={{
                                     fontWeight: "600",
                                     fontSize: "18px"
                                 }}>{t('add_submenu')} : {t('main_option')} {selectedId}</DialogTitle>
                    <CloseIcon sx={{
                        margin: '15px 15px 0 0',
                        cursor: 'pointer',
                        fontWeight: "600",
                        fontSize: "26px",
                        color: colors.addDialog[400]
                    }}
                               onClick={handleClose} />
                </Box>
                <DialogContent>
                    <Box sx={{ display: 'flex', justifyContent: 'center', padding: '10px' }}>
                        <Button
                            variant={isSubmenuPage ? "contained" : "outlined"}
                            onClick={() => setIsSubmenuPage(true)}
                            sx={{
                                marginRight: '10px',
                                textTransform: 'none',
                                color: colors.cardTitle[100],
                                backgroundColor: colors.blueAccent[700],
                                "&:hover": {
                                    backgroundColor: colors.blueAccent[800],
                                    color: colors.cardTitle[100],
                                }
                            }}
                        >
                            {t('choose_file')}
                        </Button>
                        <Button
                            variant={!isSubmenuPage ? "contained" : "outlined"}
                            onClick={() => setIsSubmenuPage(false)}
                            sx={{
                                marginRight: '10px',
                                textTransform: 'none',
                                color: colors.cardTitle[100],
                                backgroundColor: colors.blueAccent[700],
                                "&:hover": {
                                    backgroundColor: colors.blueAccent[800],
                                    color: colors.cardTitle[100],
                                }
                            }}
                        >
                            {t('add_submenu')}
                        </Button>
                        <Button
                            variant="outlined"
                            onClick={handleRefresh}
                            sx={{
                                textTransform: 'none',
                                color: colors.cardTitle[100],
                                backgroundColor: colors.blueAccent[700],
                                "&:hover": {
                                    backgroundColor: colors.blueAccent[800],
                                    color: colors.cardTitle[100],
                                }
                            }}
                        >
                            {t('reset')}
                        </Button>
                    </Box>
                    {isSubmenuPage ? (
                        <Box sx={style.submenu.secondBox}>
                            <Box sx={style.submenu.thirdBox}>
                                <Box sx={style.submenu.fourthBox}>
                                    <Typography variant={"h4"} sx={{ fontWeight: "700" }}>{t('choose_file')}</Typography>
                                    <Box sx={style.submenu.headInput}>
                                        <input
                                            id={${selectedId}_sub_main_greeting}
                                            name={${selectedId}_sub_main_greeting}
                                            type="file"
                                            accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                            style={style.submenu.inputHidden}
                                            onChange={handleFileChange}
                                            ref={ref => fileInputRefs.current[${selectedId}_sub_main_greeting] = ref}
                                        />
                                        <label htmlFor={${selectedId}_sub_main_greeting} style={{ cursor: "pointer" }}>Choose File</label>
                                    </Box>
                                </Box>
                            </Box>
                            <Box sx={{ color: colors.redAccent[500], mt: "15px" }}>
                                {error && <Typography variant="body2" sx={{ fontSize: "18px", fontWeight: "700" }}>{error}</Typography>}
                            </Box>
                        </Box>
                    ) : (
                        <Box
                            sx={style.mainMenu2.secondBox}>
                            <Box
                                sx={style.mainMenu2.thirdBox}>
                                <Box
                                    sx={style.mainMenu2.fourthBox}>
                                    <Typography variant={"h4"} sx={{fontWeight: "700"}}>Greeting</Typography>
                                    <Box
                                        sx={style.mainMenu2.headInput}>
                                        <input id="main_greeting-file" name="main_greeting" type="file"
                                               accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                               style={style.mainMenu2.inputHidden}
                                               onChange={handleFileChange}/>
                                        <label htmlFor="main_greeting-file" style={{cursor: "pointer"}}>Choose
                                            File</label>
                                    </Box>
                                </Box>
                                <Box sx={{display: "flex", alignItems: "center"}}>
                                    <Typography variant={"h4"} sx={{fontWeight: "700"}}>Goodbye</Typography>
                                    <Box
                                        sx={style.mainMenu2.headInput}>
                                        <input id="main_goodbye-file" name="main_goodbye" type="file"
                                               accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                               style={style.mainMenu2.inputHidden}
                                               onChange={handleFileChange}/>
                                        <label htmlFor="main_goodbye-file" style={{cursor: "pointer"}}>Choose
                                            File</label>
                                    </Box>
                                </Box>
                                <Box sx={{display: "flex", alignItems: "center"}}>
                                    <Typography variant={"h4"} sx={{fontWeight: "700"}}>Invalid</Typography>
                                    <Box
                                        sx={style.mainMenu2.headInput}>
                                        <input id="main_invalid-file" name="main_invalid" type="file"
                                               accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                               style={style.mainMenu2.inputHidden}
                                               onChange={handleFileChange}/>
                                        <label htmlFor="main_invalid-file" style={{cursor: "pointer"}}>Choose
                                            File</label>
                                    </Box>
                                </Box>
                            </Box>
                            <Box sx={style.mainMenu2.fifthBox}>
                                {[1, 2, 3].map((number) => (
                                    <Box
                                        sx={style.mainMenu2.bodyInput}
                                        key={number}>
                                        <Button
                                            id={${number}-file}
                                            name={${number}}
                                            style={style.mainMenu2.buttonFonSize}
                                            onClick={() => openDialog(${number})}
                                        >
                                            {number}
                                        </Button>
                                    </Box>
                                ))}
                            </Box>
                            <Box sx={style.mainMenu2.sixthBox}>
                                {[4, 5, 6].map((number) => (
                                    <Box
                                        sx={style.mainMenu2.bodyInput}
                                        key={number}>
                                        <Button
                                            id={${number}-file}
                                            name={${number}}
                                            style={style.mainMenu2.buttonFonSize}
                                            onClick={() => openDialog(${number})}
                                        >
                                            {number}
                                        </Button>
                                    </Box>
                                ))}
                            </Box>
                            <Box
                                sx={style.mainMenu2.seventhBox}>
                                {[7, 8, 9].map((number) => (
                                    <Box
                                        sx={style.mainMenu2.bodyInput}
                                        key={number}>
                                        <Button
                                            id={${number}-file}
                                            name={${number}}
                                            style={style.mainMenu2.buttonFonSize}
                                            onClick={() => openDialog(${number})}
                                        >
                                            {number}
                                        </Button>
                                    </Box>
                                ))}
                            </Box>
                            <Box sx={style.mainMenu2.eighthBox}>
                                <Box
                                    sx={style.mainMenu2.bodyInput}>
                                    <Button
                                        id={0-file}
                                        name={0}
                                        style={style.mainMenu2.buttonFonSize}
                                        onClick={() => openDialog(0)}
                                    >
                                        0
                                    </Button>
                                </Box>
                            </Box>
                            <Box sx={{color: colors.redAccent[500], mt: "15px"}}>
                                {error && <Typography variant="body2" sx={{
                                    fontSize: "18px",
                                    fontWeight: "700",
                                }}>{error}</Typography>}
                            </Box>
                        </Box>

                    )}
                </DialogContent>
            </Dialog>
            <AddMenuItem
                open={dialogOpen}
                handleClose={closeDialog}
                selectedId={localSelectedId}
                files={files}
                handleFileDataChange={handleFileDataChange}
                onReset={handleRefresh} // Updated function reference
            />
        </Box>
    );
};

export default AddMenuItemTest;
bununda sub menusu olsun 
import React, { useState, useRef } from 'react';
import {Box, Button, Dialog, DialogContent, DialogTitle, Typography} from '@mui/material';
import CloseIcon from '@mui/icons-material/Close';
import {useTranslation} from "react-i18next";
import {tokens} from "../../theme";
import useMediaQuery from "@mui/material/useMediaQuery";
import {useTheme} from "@mui/material/styles";
import useStyles1 from "../../assets/style/style";
import useStyles2 from "../../assets/style/style2";

const AddMenuItem = ({open, handleClose, selectedId, handleFileDataChange, onReset }) => {
    const theme1 = useTheme();
    const colors = tokens(theme1.palette.mode);
    const {t} = useTranslation();
    const fullScreen = useMediaQuery(theme1.breakpoints.down('md'));
    const [isSubmenuPage, setIsSubmenuPage] = useState(true);
    const [error, setError] = useState('');
    const fileInputRefs = useRef({});
    const MAX_FILE_SIZE_MB = 5;
    const isMaxWidth1280 = useMediaQuery("(max-width:1280px)");
    const styles1 = useStyles1();
    const styles2 = useStyles2();
    const style = isMaxWidth1280 ? styles2 : styles1;


    const handleFileChange = (e) => {
        const { name, files } = e.target;
        const file = files[0];
        const fileSizeMB = (file.size / (1024 * 1024)).toFixed(2);
        if (file.size > MAX_FILE_SIZE_MB * 1024 * 1024) {
            const errorMessage = `${t('max_size_limit_1')} ${MAX_FILE_SIZE_MB} ${t('max_size_limit_2')} ${t('max_size_limit_3')} ${fileSizeMB} MB.`;
            setError(errorMessage);
            return;
        } else {
            setError('');
            handleFileDataChange(name, file);
        }
    };
    const handleRefresh = () => {
        Object.keys(fileInputRefs.current).forEach(key => {
            if (fileInputRefs.current[key]) {
                fileInputRefs.current[key].value = '';
            }
        });
        setError('');
        if (onReset) {
            onReset();
        }
    };

    return (
        <Box>
            <Dialog
                fullScreen={fullScreen}
                open={open}
                onClose={handleClose}
                aria-labelledby="responsive-dialog-title"
                PaperProps={{
                    sx: {
                        maxWidth:'800px',
                        width:'800px',
                        maxHeight: "750px",
                        height:isMaxWidth1280? '550px':"750px",
                    },
                }}
            >
                <Box sx={{
                    display: 'flex',
                    justifyContent: 'space-between',
                    backgroundColor: colors.addDialog[100],
                    color: colors.addDialog[400]
                }}>
                    <DialogTitle id="responsive-dialog-title"
                                 sx={{
                                     fontWeight:"600",
                                     fontSize:"18px"
                                 }}>{t('add_submenu_to_submenu')} : {t('sub_option')} {selectedId}</DialogTitle>
                    <CloseIcon sx={{
                        margin: '15px 15px 0 0',
                        cursor: 'pointer',
                        fontWeight: "600",
                        fontSize:"26px",
                        color: colors.addDialog[400]
                    }}
                               onClick={handleClose}/>
                </Box>
                <DialogContent>
                    <Box sx={{display: 'flex', justifyContent: 'center', padding: '10px'}}>
                        <Button
                            variant={isSubmenuPage ? "contained" : "outlined"}
                            onClick={() => setIsSubmenuPage(true)}
                            sx={{marginRight: '10px', textTransform: 'none', color: colors.cardTitle[100], backgroundColor: colors.blueAccent[700],
                                "&:hover": {
                                    backgroundColor: colors.blueAccent[800],
                                    color: colors.cardTitle[100],
                                }

                            }}
                        >
                            {t('choose_file')}
                        </Button>
                        <Button
                            variant={!isSubmenuPage ? "contained" : "outlined"}
                            onClick={() => setIsSubmenuPage(false)}
                            sx={{marginRight: '10px',textTransform: 'none', color: colors.cardTitle[100], backgroundColor: colors.blueAccent[700],
                                "&:hover": {
                                    backgroundColor: colors.blueAccent[800],
                                    color: colors.cardTitle[100],
                                }
                            }}
                        >
                            {t('add_submenu')}
                        </Button>
                        <Button
                            variant="outlined"
                            onClick={handleRefresh}
                            sx={{
                                textTransform: 'none',
                                color: colors.cardTitle[100],
                                backgroundColor: colors.blueAccent[700],
                                "&:hover": {
                                    backgroundColor: colors.blueAccent[800],
                                    color: colors.cardTitle[100],
                                }
                            }}
                        >
                            {t('reset')}
                        </Button>
                    </Box>
                    {isSubmenuPage ? (
                            <Box sx={style.submenu.secondBox}>
                                <Box sx={style.submenu.thirdBox}>
                                    <Box sx={style.submenu.fourthBox}>
                                        <Typography variant={"h4"}
                                                    sx={{fontWeight: "700",}}>{t('choose_file')}</Typography>
                                        <Box sx={style.submenu.headInput}>
                                            <input id={`${selectedId}`} name={`${selectedId}`} type="file"
                                                    accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                                   style={style.submenu.inputHidden} onChange={handleFileChange}/>
                                            <label htmlFor={`${selectedId}`} style={{cursor: "pointer"}}>Choose
                                                File</label>
                                        </Box>
                                    </Box>
                                </Box>
                                <Box sx={{color: colors.redAccent[500] ,mt:"15px"}}>
                                    {error && <Typography variant="body2" sx={{fontSize:"18px",fontWeight:"700",}}>{error}</Typography>}
                                </Box>
                            </Box>
                    ) : (
                        <Box sx={style.submenu.secondBox}>
                            <Box sx={style.submenu.thirdBox}>
                                <Box sx={style.submenu.fourthBox}>
                                    <Typography variant={"h4"} sx={{fontWeight: "700"}}>Greeting</Typography>
                                    <Box sx={style.submenu.headInput}>
                                        <input id="sub_greeting-file" name={`${selectedId}_sub_greeting`} type="file"
                                               accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                               style={style.submenu.inputHidden} onChange={handleFileChange}/>
                                        <label htmlFor="sub_greeting-file" style={{cursor: "pointer"}}>Choose
                                            File</label>
                                    </Box>
                                </Box>
                                <Box sx={{display: "flex", alignItems: "center"}}>
                                    <Typography variant={"h4"} sx={{fontWeight: "700"}}>Goodbye</Typography>
                                    <Box sx={style.submenu.headInput}>
                                        <input id="sub_goodbye-file" name={`${selectedId}_sub_goodbye`} type="file"
                                                accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                               style={style.submenu.inputHidden} onChange={handleFileChange}/>
                                        <label htmlFor="sub_goodbye-file" style={{cursor: "pointer"}}>Choose
                                            File</label>
                                    </Box>
                                </Box>
                                <Box sx={{display: "flex", alignItems: "center"}}>
                                    <Typography variant={"h4"} sx={{fontWeight: "700"}}>Invalid</Typography>
                                    <Box
                                        sx={style.submenu.headInput}>
                                        <input id="sub_invalid-file" name={`${selectedId}_sub_invalid`} type="file"
                                                accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                               style={style.submenu.inputHidden} onChange={handleFileChange}/>
                                        <label htmlFor="sub_invalid-file" style={{cursor: "pointer"}}>Choose
                                            File</label>
                                    </Box>
                                </Box>
                            </Box>
                            <Box sx={style.submenu.fifthBox}>
                                <Box sx={style.submenu.bodyInput}>
                                    <input id="sub_1-file" name={`${selectedId}_sub_1`} type="file"
                                            accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                           style={style.submenu.inputHidden} onChange={handleFileChange}/>
                                    <label htmlFor="sub_1-file" style={style.submenu.bodyLabel}>1</label>
                                </Box>
                                <Box sx={style.submenu.bodyInput}>
                                    <input id="sub_2-file" name={`${selectedId}_sub_2`} type="file"
                                            accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                           style={style.submenu.inputHidden} onChange={handleFileChange}/>
                                    <label htmlFor="sub_2-file" style={style.submenu.bodyLabel}>2</label>
                                </Box>
                                <Box sx={style.submenu.bodyInput}>
                                    <input id="sub_3-file" name={`${selectedId}_sub_3`} type="file"
                                            accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                           style={style.submenu.inputHidden} onChange={handleFileChange}/>
                                    <label htmlFor="sub_3-file" style={style.submenu.bodyLabel}>3</label>
                                </Box>
                            </Box>
                            <Box sx={style.submenu.sixthBox}>
                                <Box sx={style.submenu.bodyInput}>
                                    <input id="sub_4-file" name={`${selectedId}_sub_4`} type="file"
                                            accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                           style={style.submenu.inputHidden} onChange={handleFileChange}/>
                                    <label htmlFor="sub_4-file" style={style.submenu.bodyLabel}>4</label>
                                </Box>
                                <Box sx={style.submenu.bodyInput}>
                                    <input id="sub_5-file" name={`${selectedId}_sub_5`} type="file"
                                            accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                           style={style.submenu.inputHidden} onChange={handleFileChange}/>
                                    <label htmlFor="sub_5-file" style={style.submenu.bodyLabel}>5</label>
                                </Box>
                                <Box sx={style.submenu.bodyInput}>
                                    <input id="sub_6-file" name={`${selectedId}_sub_6`} type="file"
                                            accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                           style={style.submenu.inputHidden} onChange={handleFileChange}/>
                                    <label htmlFor="sub_6-file" style={style.submenu.bodyLabel}>6</label>
                                </Box>
                            </Box>
                            <Box sx={style.submenu.seventhBox}>
                                <Box sx={style.submenu.bodyInput}>
                                    <input id="sub_7-file" name={`${selectedId}_sub_7`} type="file"
                                            accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                           style={style.submenu.inputHidden} onChange={handleFileChange}/>
                                    <label htmlFor="sub_7-file" style={style.submenu.bodyLabel}>7</label>
                                </Box>
                                <Box sx={style.submenu.bodyInput}>
                                    <input id="sub_8-file" name={`${selectedId}_sub_8`} type="file"
                                            accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                           style={style.submenu.inputHidden} onChange={handleFileChange}/>
                                    <label htmlFor="sub_8-file" style={style.submenu.bodyLabel}>8</label>
                                </Box>
                                <Box sx={style.submenu.bodyInput}>
                                    <input id="sub_9-file" name={`${selectedId}_sub_9`} type="file"
                                            accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                           style={style.submenu.inputHidden} onChange={handleFileChange}/>
                                    <label htmlFor="sub_9-file" style={style.submenu.bodyLabel}>9</label>
                                </Box>
                            </Box>
                            <Box sx={style.submenu.eighthBox}>
                                <Box sx={style.submenu.bodyInput}>
                                    <input id="sub_0-file" name={`${selectedId}_sub_0`} type="file"
                                            accept="audio/mp3,audio/mpeg,audio/x-wav,audio/wav,audio/ogg,audio/x-ms-wma"
                                           style={style.submenu.inputHidden} onChange={handleFileChange}/>
                                    <label htmlFor="sub_0-file" style={style.submenu.bodyLabel}>0</label>
                                </Box>
                            </Box>
                            <Box sx={{color: colors.redAccent[500] ,mt:"15px"}}>
                                {error && <Typography variant="body2" sx={{fontSize:"18px",fontWeight:"700",}}>{error}</Typography>}
                            </Box>
                        </Box>


                    )}
                </DialogContent>
            </Dialog>
        </Box>
    );
};

export default AddMenuItem;
